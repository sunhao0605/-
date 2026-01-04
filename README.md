<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我们的爱情故事地图</title>
    <link rel="stylesheet" href="https://unpkg.com/element-plus/dist/index.css" />
    <style>
        body, html {
            margin: 0;
            padding: 0;
            height: 100%;
            width: 100%;
            font-family: sans-serif;
            overflow: hidden;
        }
        #app {
            display: flex;
            height: 100%;
        }
        .timeline-container {
            width: 300px;
            padding: 20px;
            background-color: #f7f7f7;
            overflow-y: auto;
            border-right: 1px solid #eee;
        }
        .main-content {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
        }
        #map-container {
            flex-grow: 2;
            height: 100%;
            min-height: 400px;
            background-color: #e8e8e8;
        }
        .photo-display {
            padding: 20px;
            text-align: center;
            border-top: 1px solid #eee;
        }
        .photo-display img {
            max-width: 100%;
            max-height: 300px;
            cursor: pointer;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        }
        .initial-message {
            padding: 40px;
            color: #888;
        }
        .enlarged-img {
            width: 100%;
        }
        /* 高亮当前选中的时间轴项 */
        .el-timeline-item.is-active .el-timeline-item__content {
            font-weight: bold;
            color: #409EFF;
        }
        .el-timeline-item.is-active .el-timeline-item__dot {
            background-color: #409EFF;
        }
    </style>
</head>
<body>
    <div id="app">
        <div class="timeline-container">
            <el-timeline>
                <el-timeline-item
                    v-for="(event, index) in events"
                    :key="index"
                    :timestamp="event.date"
                    placement="right"
                    @click.native="handleClick(index)"
                    :class="{ 'is-active': currentIndex === index }"
                >
                    {{ event.title }}
                </el-timeline-item>
            </el-timeline>
        </div>
        <div class="main-content">
            <div id="map-container"></div>
            <div class="photo-display">
                <div v-if="currentEvent">
                    <h3>{{ currentEvent.title }}</h3>
                    <p>{{ currentEvent.description }}</p>
                    <img :src="currentEvent.imageUrl" @click="dialogVisible = true" @error="handleImageError" alt="故事照片" />
                </div>
                <div v-else class="initial-message">
                    点击左侧时间轴，开始我们的故事...
                </div>
            </div>
        </div>
        <el-dialog v-model="dialogVisible" title="美好瞬间" width="80%" append-to-body>
            <img :src="currentEvent.imageUrl" class="enlarged-img" @error="handleImageError" alt="放大的故事照片" />
        </el-dialog>
    </div>

    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://unpkg.com/element-plus/dist/index.full.js"></script>
    <script type="text/javascript" src="https://webapi.amap.com/maps?v=2.0&key=5fdd01771aeb6d70afbf04df394b940f&securityJsCode=8bf3ac670a3443a8477ebf6734d6f721"></script>

    <script>
        const { createApp, ref, onMounted } = Vue;

        createApp({
            setup() {
                const events = ref([
                    {
                        date: '2008-09-14',
                        title: '初次相遇',
                        description: '在高中相遇，一个阳光明媚的日子...',
                        imageUrl: 'https://img.remit.ee/api/file/BQACAgUAAyEGAASHRsPbAAEPidNpWkI0mcL86nt_WDCOxKvs8-wbyAACGBwAAkKH2FbKUeEa43Uw6DgE.jpg', 
                        position: [124.826, 46.047] // [经度, 纬度] 格式
                    },
                    {
                        date: '2016-12-20',
                        title: '第一次去你家',
                        description: '寒冷的冬天，第一次踏上鹤岗的土地。',
                        imageUrl: 'https://img.remit.ee/api/file/BQACAgUAAyEGAASHRsPbAAEPihFpWkqzDmaehy5W-brLOlTw4tpXyQACgRwAAkKH2FZlhEGGwJTX0TgE.jpg',
                        position: [130.309,47.363] // [经度, 纬度] 格式
                    },
                    // 你可以在这里继续添加更多的故事
                    // {
                    //     date: 'YYYY-MM-DD',
                    //     title: '事件标题',
                    //     description: '事件描述...',
                    //     imageUrl: '图片URL',
                    //     position: [经度, 纬度]
                    // }
                ]);

                const currentIndex = ref(-1);
                const currentEvent = ref(null);
                const dialogVisible = ref(false);
                let map = null;
                let markers = [];

                const handleClick = (index) => {
                    if (index < 0 || index >= events.value.length) return;
                    
                    currentIndex.value = index;
                    currentEvent.value = events.value[index];
                    
                    const targetEvent = events.value[index];
                    const targetPosition = targetEvent.position;

                    if (!targetPosition || !map) return;

                    // 移动地图到目标位置
                    map.setZoomAndCenter(15, targetPosition);

                    // 更新所有标记点的图标，将选中的标记点高亮
                    markers.forEach((marker, i) => {
                        const iconPath = (i === index) 
                            ? 'https://webapi.amap.com/theme/v1.3/markers/n/mark_b.png' 
                            : 'https://webapi.amap.com/theme/v1.3/markers/n/mark_a.png';
                        marker.setIcon(iconPath);
                    });
                };
                
                const handleImageError = (event) => {
                    // 如果图片加载失败，显示一个占位图
                    event.target.src = 'https://placehold.co/600x400/e8f4f8/4299e1?text=Image+Not+Found';
                };

                onMounted(() => {
                    // 确保高德地图API加载完成
                    if (window.AMap) {
                        initMap();
                    } else {
                        // 如果API还未加载，轮询等待
                        const interval = setInterval(() => {
                            if (window.AMap) {
                                clearInterval(interval);
                                initMap();
                            }
                        }, 100);
                    }
                });

                const initMap = () => {
                    // 初始化地图
                    map = new AMap.Map('map-container', {
                        zoom: 4,
                        center: [118, 34],
                        resizeEnable: true
                    });

                    // 为每个事件创建标记点
                    events.value.forEach((event, index) => {
                        if (!event.position) {
                            console.warn(`事件 "${event.title}" 缺少坐标信息，已跳过。`);
                            return;
                        }
                        const marker = new AMap.Marker({
                            position: event.position,
                            title: event.title,
                            icon: 'https://webapi.amap.com/theme/v1.3/markers/n/mark_a.png'
                        });
                        
                        // 为标记点绑定点击事件
                        marker.on('click', () => handleClick(index));
                        
                        markers.push(marker);
                    });
                    
                    // 将所有标记点添加到地图上
                    map.addMarkers(markers);

                    // 如果有事件，默认选中第一个
                    if (events.value.length > 0) {
                        handleClick(0);
                    }
                };

                return {
                    events,
                    currentIndex,
                    currentEvent,
                    dialogVisible,
                    handleClick,
                    handleImageError
                };
            }
        }).use(ElementPlus).mount('#app');
    </script>
</body>
</html>
