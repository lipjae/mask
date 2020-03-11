<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md">
      <q-input v-model="searchKeyword" label="주소를 입력해 주세요." style="margin-bottom: 20px;">
        <template v-slot:append>
          <q-icon v-if="searchKeyword !== ''" name="close" @click="searchKeyword = ''" class="cursor-pointer" />
          <q-icon name="search" @click="searchAddress()"/>
        </template>
      </q-input>
      <q-select v-model="selectAddress" :options="addressList" label="주소를 선택해주세요."/>
      <div id="map" style="width:600px;height:600px;"></div>
    </div>
    

    
  </q-page>
</template>

<script>

import kakao from '../boot/kakao'
import _ from 'lodash'

export default {
  name: 'PageIndex',
  data() {
    return {
      searchKeyword: '',
      selectAddress: null,
      addressList: []
    }
  },
  watch: {
    async selectAddress () {
      this.setMap()
    }
  },
  async mounted() {
    
    // let getCurrentLocation = function () {
    //   return new Promise(function (resolve, reject) {
    //     navigator.geolocation.getCurrentPosition(resolve, reject);
    //   })
    // }
  },
  methods: {
    async getStore (lat, lng) {
      try {
        const stores = await this.$axios.get('https://8oi9s0nnth.apigw.ntruss.com/corona19-masks/v1/storesByGeo/json?',{
          params: {
            lat: lat,
            lng: lng,
            m: 3000
          }
        })  
        return stores.data.stores
      } catch (error) {
        alert('약국 정보를 가져올수 없습니다.')
      }
    },
    async searchAddress () {     
      try {
        let query = `query=${this.searchKeyword}`
        let address = await this.$axios.get(`https://dapi.kakao.com/v2/local/search/address.json?${query}`,{
          headers: {
            Authorization: 'KakaoAK d62cabb2520e0a6ecb26b19405cee1da'
          }
        })
        let addrList = []
        _.each(address.data.documents, function(value, key){
          addrList.push({
            label: value.address_name,
            value: value
          })
        })
        this.addressList = addrList
      } catch (error) {
        throw error  
      }
    },
    async setMap () {
      let lat = this.selectAddress.value.y
      let lng = this.selectAddress.value.x
      console.log(lat, lng)
      var container = document.getElementById('map'); //지도를 담을 영역의 DOM 레퍼런스
      var options = { //지도를 생성할 때 필요한 기본 옵션
        center: new kakao.maps.LatLng(lat, lng), //지도의 중심좌표.
        level: 3 //지도의 레벨(확대, 축소 정도)
      };
      var map = new kakao.maps.Map(container, options); //지도 생성 및 객체 리턴
      
      const stores = await this.getStore(lat, lng)
      let positions = []
      _.each(stores, (value, key) => {
        positions.push({
          content: `<div>
              <p>${value.name}</p>
              <p>재고상태: ${value.remain_stat}</p>
              <p>업데이트 시간: ${value.stock_at}</p>
            </div>`,
          latlng: new kakao.maps.LatLng(value.lat, value.lng)
        })
      })  

      // 마커 이미지의 이미지 주소입니다
      var imageSrc = "http://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png"; 
          
      for (var i = 0; i < positions.length; i ++) {
          
        // 마커 이미지의 이미지 크기 입니다
        var imageSize = new kakao.maps.Size(24, 35); 
        
        // 마커 이미지를 생성합니다    
        var markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize); 
        
        // 마커를 생성합니다
        var marker = new kakao.maps.Marker({
            map: map, // 마커를 표시할 지도
            position: positions[i].latlng, // 마커를 표시할 위치
            // title : positions[i].title, // 마커의 타이틀, 마커에 마우스를 올리면 타이틀이 표시됩니다
            // image : markerImage // 마커 이미지 
        });

          // 마커에 표시할 인포윈도우를 생성합니다 
        var infowindow = new kakao.maps.InfoWindow({
            content: positions[i].content // 인포윈도우에 표시할 내용
        });

        // 마커에 mouseover 이벤트와 mouseout 이벤트를 등록합니다
        // 이벤트 리스너로는 클로저를 만들어 등록합니다 
        // for문에서 클로저를 만들어 주지 않으면 마지막 마커에만 이벤트가 등록됩니다
        kakao.maps.event.addListener(marker, 'mouseover', makeOverListener(map, marker, infowindow));
        kakao.maps.event.addListener(marker, 'mouseout', makeOutListener(infowindow));
      }
    

      // 인포윈도우를 표시하는 클로저를 만드는 함수입니다 
      function makeOverListener(map, marker, infowindow) {
          return function() {
              infowindow.open(map, marker);
          };
      }

      // 인포윈도우를 닫는 클로저를 만드는 함수입니다 
      function makeOutListener(infowindow) {
          return function() {
              infowindow.close();
          };
      }
    }
  }
}
</script>
