
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>이천시 스마트 의료 지도</title>

<script type="text/javascript"
src="https://dapi.kakao.com/v2/maps/sdk.js?appkey=4106a0cc81c61ef4889c6d4acf55a48d&libraries=services"></script>

<style>

*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    font-family:'Pretendard',sans-serif;
    background:#f8fafc;
    color:#1e293b;
}

header{
    background:linear-gradient(135deg,#38bdf8,#0284c7);
    color:white;
    text-align:center;
    padding:22px;
}

header h1{
    font-size:1.5rem;
    margin-bottom:5px;
}

.filter-bar{
    background:white;
    padding:14px;
    display:flex;
    justify-content:center;
    gap:8px;
    flex-wrap:wrap;
    border-bottom:1px solid #e2e8f0;
}

.filter-btn{
    border:none;
    padding:8px 15px;
    border-radius:20px;
    background:#e0f2fe;
    cursor:pointer;
    font-size:13px;
    transition:0.2s;
}

.filter-btn.active{
    background:#0284c7;
    color:white;
    font-weight:bold;
}

.main{
    width:98%;
    margin:15px auto;
    display:grid;
    grid-template-columns:320px 1fr 320px;
    gap:15px;
}

.panel{
    background:white;
    border-radius:18px;
    padding:20px;
    height:82vh;
    display:flex;
    flex-direction:column;
    box-shadow:0 4px 12px rgba(0,0,0,0.08);
}

.map-wrap{
    position:relative;
}

#map{
    width:100%;
    height:82vh;
    border-radius:18px;
    box-shadow:0 4px 12px rgba(0,0,0,0.08);
}

.route-cancel-btn{
    position:absolute;
    top:15px;
    right:15px;
    z-index:10;
    border:none;
    background:#ef4444;
    color:white;
    padding:10px 14px;
    border-radius:12px;
    font-weight:bold;
    cursor:pointer;
    display:none;
    box-shadow:0 4px 10px rgba(0,0,0,0.15);
}

.green-btn,
.blue-btn{
    width:100%;
    border:none;
    border-radius:12px;
    padding:12px;
    color:white;
    font-weight:bold;
    cursor:pointer;
    margin-bottom:10px;
}

.green-btn{
    background:#22c55e;
}

.blue-btn{
    background:#0ea5e9;
}

.input-box{
    margin-bottom:8px;
}

.input-box input{
    width:100%;
    padding:10px;
    border:1px solid #e2e8f0;
    border-radius:10px;
}

.search-input{
    width:100%;
    padding:10px;
    border:1px solid #e2e8f0;
    border-radius:10px;
    margin-bottom:15px;
}

.hospital-list{
    overflow-y:auto;
    flex:1;
}

.card{
    border:1px solid #e2e8f0;
    border-radius:14px;
    padding:14px;
    margin-bottom:10px;
    cursor:pointer;
    transition:0.2s;
}

.card:hover{
    background:#f0f9ff;
}

.iw-box{
    width:240px;
    padding:10px;
}

.iw-title{
    font-size:15px;
    color:#0284c7;
    font-weight:bold;
}

.iw-phone{
    color:#ef4444;
    font-weight:bold;
    display:block;
    margin-top:8px;
}

.iw-btns{
    display:flex;
    gap:5px;
    margin-top:10px;
    border-top:1px solid #eee;
    padding-top:8px;
}

.iw-btns button{
    flex:1;
    border:none;
    border-radius:6px;
    padding:6px;
    color:white;
    cursor:pointer;
    font-size:11px;
    font-weight:bold;
}

@keyframes pulse{

    0%{
        transform:scale(1);
    }

    50%{
        transform:scale(1.15);
    }

    100%{
        transform:scale(1);
    }

}

</style>
</head>

<body>

<header>
    <h1>이천시 스마트 의료 지도</h1>
    <p>실시간 병원 탐색 시스템</p>
</header>

<div class="filter-bar">

<button class="filter-btn active"
onclick="setCategory('전체',event)">전체</button>

<button class="filter-btn"
onclick="setCategory('내과',event)">내과</button>

<button class="filter-btn"
onclick="setCategory('소아과',event)">소아과</button>

<button class="filter-btn"
onclick="setCategory('정형외과',event)">정형외과</button>

<button class="filter-btn"
onclick="setCategory('치과',event)">치과</button>

<button class="filter-btn"
onclick="setCategory('한의원',event)">한의원</button>

<button class="filter-btn"
onclick="setCategory('산부인과',event)">산부인과</button>

<button class="filter-btn"
onclick="setCategory('안과',event)">안과</button>

</div>

<div class="main">

<div class="panel">

<button class="green-btn"
onclick="moveMyLocation()">
📍 내 위치 찾기
</button>

<div class="input-box">
<input type="text" id="newName" placeholder="병원 이름">
</div>

<div class="input-box">
<input type="text" id="newType" placeholder="진료과">
</div>

<div class="input-box">
<input type="text" id="newPhone" placeholder="전화번호">
</div>

<div class="input-box">
<input type="text" id="newInfo" placeholder="진료 정보">
</div>

<button class="blue-btn"
onclick="addHospital()">
➕ 병원 추가
</button>

</div>

<div class="map-wrap">

<button
class="route-cancel-btn"
id="routeCancelBtn"
onclick="clearRoute()">
❌ 길찾기 취소
</button>

<div id="map"></div>

</div>

<div class="panel">

<input type="text"
id="searchInput"
class="search-input"
placeholder="병원 검색..."
onkeyup="renderHospitalList()">

<div id="hospitalList"
class="hospital-list"></div>

</div>

</div>

<script>

const map =
new kakao.maps.Map(

    document.getElementById('map'),

    {
        center:new kakao.maps.LatLng(37.2799,127.4423),
        level:6
    }

);

const places =
new kakao.maps.services.Places();

let hospitals = [];
let markers = [];
let markerMap = {};

let currentFilter = '전체';

let currentInfoWindow = null;
let selectedPolyline = null;

let myLat = null;
let myLng = null;

const keywords = [

    '이천시 병원',
    '이천시 내과',
    '이천시 소아과',
    '이천시 치과',
    '이천시 정형외과',
    '이천시 한의원',
    '이천시 산부인과',
    '이천시 안과'

];

let done = 0;

keywords.forEach(keyword=>{

    places.keywordSearch(

        keyword,

        (data,status)=>{

            if(status === kakao.maps.services.Status.OK){

                data.forEach(place=>{

                    const exists =
                    hospitals.some(h=>
                        h.name === place.place_name
                    );

                    if(!exists){

                        hospitals.push({

                            name:place.place_name,

                            type:getType(
                                place.category_name
                            ),

                            phone:
                            place.phone ||
                            '번호 없음',

                            info:
                            place.category_name,

                            lat:Number(place.y),

                            lng:Number(place.x)

                        });

                    }

                });

            }

            done++;

            if(done === keywords.length){

                renderHospitals();

            }

        }

    );

});

function getType(category){

    if(category.includes('내과'))
    return '내과';

    if(category.includes('소아'))
    return '소아과';

    if(category.includes('정형'))
    return '정형외과';

    if(category.includes('치과'))
    return '치과';

    if(category.includes('한의'))
    return '한의원';

    if(category.includes('산부인'))
    return '산부인과';

    if(category.includes('안과'))
    return '안과';

    return '기타';

}

function moveMyLocation(){

navigator.geolocation.getCurrentPosition(

    pos=>{

        myLat = pos.coords.latitude;
        myLng = pos.coords.longitude;

        const loc =
        new kakao.maps.LatLng(myLat,myLng);

        map.panTo(loc);

        map.setLevel(2);

        if(window.myMarker){

            window.myMarker.setMap(null);

        }

        window.myMarker =
        new kakao.maps.CustomOverlay({

            position:loc,

            content:`

            <div style="
                width:20px;
                height:20px;
                background:#22c55e;
                border:4px solid white;
                border-radius:50%;
                box-shadow:
                0 0 0 8px rgba(34,197,94,0.25),
                0 0 15px rgba(0,0,0,0.25);
                animation:pulse 1.7s infinite;
            "></div>

            `,

            yAnchor:0.5

        });

        window.myMarker.setMap(map);

        renderHospitalList();

    }

);

}

function setCategory(type,event){

    currentFilter = type;

    document.querySelectorAll('.filter-btn')
    .forEach(btn=>
        btn.classList.remove('active')
    );

    event.target.classList.add('active');

    renderHospitals();

}

function getDistance(lat1,lng1,lat2,lng2){

    const R = 6371;

    const dLat = (lat2-lat1) * Math.PI / 180;
    const dLng = (lng2-lng1) * Math.PI / 180;

    const a =
    Math.sin(dLat/2) * Math.sin(dLat/2) +
    Math.cos(lat1 * Math.PI/180) *
    Math.cos(lat2 * Math.PI/180) *
    Math.sin(dLng/2) * Math.sin(dLng/2);

    const c = 2 * Math.atan2(Math.sqrt(a),Math.sqrt(1-a));

    return R * c;

}

function renderHospitals(){

    markers.forEach(m=>m.setMap(null));

    markers = [];
    markerMap = {};

    hospitals.forEach((h,index)=>{

        if(
            currentFilter !== '전체' &&
            h.type !== currentFilter
        ){
            return;
        }

        const pos =
        new kakao.maps.LatLng(h.lat,h.lng);

        const marker =
        new kakao.maps.Marker({

            position:pos,
            map:map,
            draggable:true

        });

        const svg = `
        <svg xmlns="http://www.w3.org/2000/svg"
        width="22"
        height="22">

        <circle
        cx="11"
        cy="11"
        r="8"
        fill="#0ea5e9"
        stroke="white"
        stroke-width="4"/>

        </svg>
        `;

        const markerImage =
        new kakao.maps.MarkerImage(

            'data:image/svg+xml;charset=utf-8,' +
            encodeURIComponent(svg),

            new kakao.maps.Size(22,22)

        );

        marker.setImage(markerImage);

        markers.push(marker);

        markerMap[h.name] = {
            marker,
            position:pos
        };

        kakao.maps.event.addListener(

            marker,

            'dragend',

            ()=>{

                const newPos =
                marker.getPosition();

                const oldPos =
                new kakao.maps.LatLng(
                    h.lat,
                    h.lng
                );

                if(confirm('위치를 저장할까요?')){

                    h.lat = newPos.getLat();
                    h.lng = newPos.getLng();

                }

                else{

                    marker.setPosition(oldPos);

                }

            }

        );

        kakao.maps.event.addListener(

            marker,

            'click',

            ()=>{

                if(
                    currentInfoWindow &&
                    currentInfoWindow.marker === marker
                ){

                    currentInfoWindow.close();
                    currentInfoWindow = null;
                    return;

                }

                if(currentInfoWindow){

                    currentInfoWindow.close();

                }

                const content = `

                <div class="iw-box">

                    <div class="iw-title">
                    ${h.name}
                    </div>

                    <div style="
                    margin-top:5px;
                    font-size:12px;
                    color:#64748b;
                    ">
                    ${h.type}
                    </div>

                    <div style="
                    margin-top:6px;
                    font-size:12px;
                    ">
                    ${h.info}
                    </div>

                    <span class="iw-phone">
                    ☎ ${h.phone}
                    </span>

                    <div class="iw-btns">

                        <button
                        style="background:#0ea5e9;"
                        onclick="editHospital(${index})">
                        수정
                        </button>

                        <button
                        style="background:#94a3b8;"
                        onclick="deleteHospital(${index})">
                        삭제
                        </button>

                        <button
                        style="background:#22c55e;"
                        onclick="drawRoute(${h.lat},${h.lng})">
                        길찾기
                        </button>

                    </div>

                </div>

                `;

                const iw =
                new kakao.maps.InfoWindow({

                    content:content,
                    removable:true

                });

                iw.marker = marker;

                iw.open(map,marker);

                currentInfoWindow = iw;

            }

        );

    });

    renderHospitalList();

}

function renderHospitalList(){

    const keyword =
    document
    .getElementById('searchInput')
    .value
    .toLowerCase();

    const list =
    document.getElementById(
        'hospitalList'
    );

    list.innerHTML = '';

    let filteredHospitals = hospitals.filter(h=>{

        if(
            currentFilter !== '전체' &&
            h.type !== currentFilter
        ){
            return false;
        }

        if(
            !h.name
            .toLowerCase()
            .includes(keyword)
        ){
            return false;
        }

        return true;

    });

    if(myLat !== null){

        filteredHospitals.sort((a,b)=>{

            const distA = getDistance(
                myLat,
                myLng,
                a.lat,
                a.lng
            );

            const distB = getDistance(
                myLat,
                myLng,
                b.lat,
                b.lng
            );

            return distA - distB;

        });

    }

    filteredHospitals.forEach(h=>{

        const card =
        document.createElement('div');

        card.className = 'card';

        let distanceText = '';

        if(myLat !== null){

            const dist = getDistance(
                myLat,
                myLng,
                h.lat,
                h.lng
            );

            distanceText = `
            <p style="margin-top:5px;color:#0284c7;font-size:13px;">
            📍 ${dist.toFixed(1)}km
            </p>
            `;

        }

        card.innerHTML = `

        <h4>${h.name}</h4>

        <p style="margin-top:5px;">
        ${h.type}
        </p>

        <p style="margin-top:5px;">
        ☎ ${h.phone}
        </p>

        ${distanceText}

        `;

        card.onclick = ()=>{

            const target =
            markerMap[h.name];

            if(!target) return;

            map.panTo(
                target.position
            );

            setTimeout(()=>{

                map.setLevel(1);

            },300);

            kakao.maps.event.trigger(
                target.marker,
                'click'
            );

        };

        list.appendChild(card);

    });

}

function editHospital(index){

    const h = hospitals[index];

    h.name =
    prompt('병원 이름',h.name)
    || h.name;

    h.type =
    prompt('진료과',h.type)
    || h.type;

    h.phone =
    prompt('전화번호',h.phone)
    || h.phone;

    h.info =
    prompt('진료 정보',h.info)
    || h.info;

    renderHospitals();

}

function deleteHospital(index){

    if(confirm('삭제할까요?')){

        hospitals.splice(index,1);

        renderHospitals();

    }

}

function addHospital(){

    const name =
    document.getElementById('newName').value;

    const type =
    document.getElementById('newType').value;

    const phone =
    document.getElementById('newPhone').value;

    const info =
    document.getElementById('newInfo').value;

    if(!name || !type){

        alert('병원 이름과 진료과를 입력하세요!');
        return;

    }

    hospitals.push({

        name,
        type,
        phone,
        info,
        lat:37.2799,
        lng:127.4423

    });

    renderHospitals();

}

async function drawRoute(lat,lng){

    if(myLat === null){

        alert('먼저 내 위치를 불러와주세요!');
        return;

    }

    try{

        if(selectedPolyline){

            selectedPolyline.setMap(null);

        }

        const response =
        await fetch(

`https://apis-navi.kakaomobility.com/v1/directions?origin=${myLng},${myLat}&destination=${lng},${lat}&priority=RECOMMEND`,

            {

                method:'GET',

                headers:{

                    Authorization:
                    'KakaoAK 3037f8baadf2514a8544383cbc2a1d49',

                    'Content-Type':
                    'application/json'

                }

            }

        );

        const data =
        await response.json();

        if(!data.routes){

            alert('길찾기 실패');
            return;

        }

        const roads =
        data.routes[0]
        .sections[0]
        .roads;

        let linePath = [];

        roads.forEach(road=>{

            for(
                let i=0;
                i<road.vertexes.length;
                i+=2
            ){

                linePath.push(

                    new kakao.maps.LatLng(

                        road.vertexes[i+1],
                        road.vertexes[i]

                    )

                );

            }

        });

        selectedPolyline =
        new kakao.maps.Polyline({

            path:linePath,

            strokeWeight:6,

            strokeColor:'#22c55e',

            strokeOpacity:0.9,

            strokeStyle:'solid'

        });

        selectedPolyline.setMap(map);

        document.getElementById(
            'routeCancelBtn'
        ).style.display = 'block';

        const bounds =
        new kakao.maps.LatLngBounds();

        linePath.forEach(p=>bounds.extend(p));

        map.setBounds(bounds);

    }

    catch(e){

        console.error(e);
        alert('길찾기 오류');

    }

}

function clearRoute(){

    if(selectedPolyline){

        selectedPolyline.setMap(null);
        selectedPolyline = null;

    }

    document.getElementById(
        'routeCancelBtn'
    ).style.display = 'none';

}

renderHospitals();

</script>

</body>
</html>
```
