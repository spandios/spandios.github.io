---
layout: post
title: Youtube DATA API로 크롤링한 데이터 정제
date: 2024-07-09 08:00 +0900
img_path: /assets/images/
category: etc
tags: [ youtube, api, crawling ]
---

### 유투브 채널 이미지 URL 만료 
기존 크롤링 된 유투브 영상 데이터를 사용하려니 몇몇 채널 프로필 이미지의 URL이 만료된 것을 확인했다. 

유투버들이 채널 이미지를 변경하게 되면서 기존 데이터가 만료된 것으로 보인다. 

그래서 youtube data api를 사용하여 만료된 이미지는 새로 크롤링하여 데이터를 정제하려고 한다.

먼저 크롤링한 데이터는 대략 아래와 같다.

```json
{
  "title": "유튜브 영상 제목",
  "profile_url": "유튜버 채널 프로필 이미지 url",
  "youtuber_name": "유튜버 이름",
  "youtube_code": "video id"
}
```

### 검색 API 사용해보기

먼저 문서를 보니 [검색  API](https://developers.google.com/youtube/v3/docs/search/list?hl=ko#usage)가 바로 채널 썸네일도 응답값이 있고 크롤링 된 데이터에 채널 id가 없는 현 상황에서 가장 쉽게 사용할 수 있다고 판단했다.

썸네일이 유효한지 확인하고 만약 유효하지 않으면 검색 API를 사용해 새로운 썸네일을 가져오고 데이터를 업데이트하는 방식으로 진행했다.

```javascript
// 간소 예시
async function findYoutuber(){
  return `select id, profile_url from video group by youtuber_name`
}

async function checkImageUrl(url) {
  try {
    const response = await axios.head(url);
    return response.status === 200; // 200은 정상적으로 이미지가 있는 상태
  } catch (error) {
    return false;
  }
}
```

하지만 분명 하루에 10,000 할당량으로 알고 있었는데 몇번 사용하지도 않았는데 할당량 오류가 발생했다.  

### 할당량 
그러다가 [할당량 확인](https://developers.google.com/youtube/v3/determine_quota_cost?hl=ko) 이곳을 확인하게 되었는데 검색 API는 100개의 단위로 사용량이 측정된다고 한다. 

즉, API마다 할당량 단위가 다른거였고 검색 API는 100단위이때문에 하루 100번 밖에 사용할 수 없었던 것이다. 

### channel list api
다행히 [채널 조회 API](https://developers.google.com/youtube/v3/docs/channels/list?hl=ko)는 할당량 단위가 1이기 때문에 하루에 사용하기에 충분했다. 

현재 유투버 이름만 알기 때문에 채널 조회 API에서 `forUsername`라는 쿼리 파라미터 옵션을 사용하려했지만,,, 무슨 이유인지 결과가 제대로 나오지 않았다. API에 문제가 있는 것 같았다. 결국엔 channel id를 구해야 한다!

### channel id 구하기 

 다행히 [video 조회 api](https://developers.google.com/youtube/v3/docs/videos/list?hl=ko)의 응답값에 channelId가 있고 현재 video의 id는 알고 있기 때문에 이를 사용해 채널 id를 구할 수 있었다.

```javascript
async function findChannelIdByVideoId(id) {
  const searchUrl = `https://www.googleapis.com/youtube/v3/videos?part=snippet&id=${id}&key=${API_KEY}`;

  try {
    const response = await axios.get(searchUrl);
    const searchData = response.data;
    if (searchData.items && searchData.items.length > 0) {
      return searchData.items[0].snippet['channelId'];
    } else {
      console.log(`비디오 ID :${id}로부터 채널 ID를 찾을 수 없습니다.`);
    }
  } catch (error) {

    console.log(
      `Error occurred while fetching channel ID. youtube ID :${id} , ${error}`,
    );
  }
}
```

최종적으로는 아래 코드처럼 다시 channel api에 channel id로부터 채널 프로필 이미지를 얻을 수 있었다.

```javascript

async function findChannelThumbnailById(channelId) {
  const searchUrl = `https://www.googleapis.com/youtube/v3/channels?part=snippet&id=${channelId}&key=${API_KEY}`;

  try {
    const response = await axios.get(searchUrl);
    const searchData = response.data;
    if (searchData.items && searchData.items.length > 0) {
      return searchData.items[0].snippet['thumbnails']['default']['url'];
    } else {
      console.log(`채널 ID :${channelId}로부터 썸네일 URL을 찾을 수 없습니다.`);
    }
  } catch (error) {
    console.log(
      `Error occurred while fetching channel thumbnail. channel ID :${channelId} , ${error}`,
    );
  }
}
```

  
### 꿀팁 - video 조회 API 한번에 최대 50개를 조회할 수 있다!


[video 조회 api](https://developers.google.com/youtube/v3/docs/videos/list?hl=ko)이 문서에 maxResults 파라미터 설명으로는 id로 조회시 때는 기본값 5만 적용된다고 되어 있다. 

![]({{site.url}}/assets/images/decorator.png)

뭔가 열받아서 그냥 50개로 시도해보니 잘만 작동 되었다!

아래 코드는 삭제된 영상을 찾는 예시이다.

```javascript
async function findRemovedVideo(videoIds) {
  const ids = videoIds.join(',');
  const searchUrl = `https://www.googleapis.com/youtube/v3/videos?part=snippet&id=${ids}&key=${API_KEY}&maxResults=50`;

  try {
    const response = await axios.get(searchUrl);
    const searchData = response.data;
    if (searchData.items && searchData.items.length > 0) {
      const runningVideoIds = searchData.items.map((item) => item.id);
      return videoIds.filter((id) => !runningVideoIds.includes(id));
    } else {
      return videoIds;
    }
  } catch (error) {
    console.log(error);
  }
}
```

할당량 1로 50개를 조회 할 수 있으니 조회할 데이터가 많다면 꼭 이렇게 사용하자~









 




