<template>
    <div ref="canvasContainer" class="fixed inset-0 w-screen h-screen bg-black touch-none select-none">
        <!-- 添加上升下降控制按钮 -->
        <div class="absolute bottom-8 right-8 flex flex-col gap-2 z-10">
            <button @click="moveUp" class="bg-blue-500 hover:bg-blue-600 text-white rounded-full w-12 h-12 flex items-center justify-center shadow-lg">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 15l7-7 7 7" />
                </svg>
            </button>
            <button @click="moveDown" class="bg-blue-500 hover:bg-blue-600 text-white rounded-full w-12 h-12 flex items-center justify-center shadow-lg">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
                </svg>
            </button>
        </div>
        
        <!-- 添加预设视角按钮 -->
        <div class="absolute top-4 right-4 flex gap-2 z-10">
            <button @click="setTopView" class="bg-gray-700 hover:bg-gray-600 text-white px-3 py-1 rounded shadow-lg">
                顶视图
            </button>
            <button @click="setFrontView" class="bg-gray-700 hover:bg-gray-600 text-white px-3 py-1 rounded shadow-lg">
                前视图
            </button>
            <button @click="setSideView" class="bg-gray-700 hover:bg-gray-600 text-white px-3 py-1 rounded shadow-lg">
                侧视图
            </button>
        </div>
        
        <!-- 全屏按钮 -->
        <button @click="toggleFullScreen" class="absolute bottom-8 left-8 z-10 bg-gray-700 hover:bg-gray-600 text-white rounded-full w-12 h-12 flex items-center justify-center shadow-lg">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 8V4m0 0h4M4 4l5 5m11-1V4m0 0h-4m4 0l-5 5M4 16v4m0 0h4m-4 0l5-5m11 5v-4m0 4h-4m4 0l-5-5" />
            </svg>
        </button>
    </div>
</template>

<script>
import { onMounted, ref, onBeforeUnmount } from 'vue';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
    name: 'ThreeDModel',
    setup() {
        const canvasContainer = ref(null);
        let renderer, scene, camera, controls;
        
        // 处理窗口大小变化
        const handleResize = () => {
            if (camera && renderer) {
                const width = window.innerWidth;
                const height = window.innerHeight;
                
                // 更新相机宽高比
                camera.aspect = width / height;
                camera.updateProjectionMatrix();
                
                // 更新渲染器尺寸
                renderer.setSize(width, height);
                renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // 限制像素比以提高性能
            }
        };
        
        // 处理可见性变化
        const handleVisibilityChange = () => {
            if (document.visibilityState === 'visible') {
                // 页面恢复可见时，确保大小正确
                handleResize();
            }
        };
        
        // 上升和下降的移动速度
        const moveSpeed = 0.5;
        
        // 上升函数 - 将相机向上移动
        const moveUp = () => {
            if (camera && controls) {
                // 获取相机当前位置
                const position = new THREE.Vector3();
                position.copy(camera.position);
                
                // 向上移动相机
                position.y += moveSpeed;
                camera.position.copy(position);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 下降函数 - 将相机向下移动
        const moveDown = () => {
            if (camera && controls) {
                // 获取相机当前位置
                const position = new THREE.Vector3();
                position.copy(camera.position);
                
                // 向下移动相机
                position.y -= moveSpeed;
                camera.position.copy(position);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 向前移动
        const moveForward = () => {
            if (camera && controls) {
                // 获取相机的前方向
                const direction = new THREE.Vector3();
                camera.getWorldDirection(direction);
                direction.normalize();
                
                // 向前移动相机
                camera.position.addScaledVector(direction, moveSpeed);
                controls.target.addScaledVector(direction, moveSpeed);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 向后移动
        const moveBackward = () => {
            if (camera && controls) {
                // 获取相机的前方向
                const direction = new THREE.Vector3();
                camera.getWorldDirection(direction);
                direction.normalize();
                
                // 向后移动相机
                camera.position.addScaledVector(direction, -moveSpeed);
                controls.target.addScaledVector(direction, -moveSpeed);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 向左移动
        const moveLeft = () => {
            if (camera && controls) {
                // 获取相机的右方向向量
                const right = new THREE.Vector3();
                right.crossVectors(camera.up, camera.getWorldDirection(new THREE.Vector3()));
                right.normalize();
                
                // 向左移动相机
                camera.position.addScaledVector(right, moveSpeed);
                controls.target.addScaledVector(right, moveSpeed);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 向右移动
        const moveRight = () => {
            if (camera && controls) {
                // 获取相机的右方向向量
                const right = new THREE.Vector3();
                right.crossVectors(camera.up, camera.getWorldDirection(new THREE.Vector3()));
                right.normalize();
                
                // 向右移动相机
                camera.position.addScaledVector(right, -moveSpeed);
                controls.target.addScaledVector(right, -moveSpeed);
                
                // 更新控制器
                controls.update();
            }
        };
        
        // 键盘控制处理函数
        const handleKeyDown = (event) => {
            switch (event.key) {
                case 'ArrowUp':
                case 'w':
                case 'W':
                    moveForward();
                    break;
                case 'ArrowDown':
                case 's':
                case 'S':
                    moveBackward();
                    break;
                case 'ArrowLeft':
                case 'a':
                case 'A':
                    moveLeft();
                    break;
                case 'ArrowRight':
                case 'd':
                case 'D':
                    moveRight();
                    break;
                case 'q':
                case 'Q':
                case 'PageUp':
                    moveUp();
                    break;
                case 'e':
                case 'E':
                case 'PageDown':
                    moveDown();
                    break;
            }
        };
        
        // 阻止浏览器默认的触摸行为
        const preventDefaultTouchActions = (event) => {
            event.preventDefault();
        };
        
        // 专门处理右键事件
        const handleRightClick = (event) => {
            event.preventDefault(); // 阻止浏览器默认右键菜单
            return false;
        };

        // 预设视角函数
        const setTopView = () => {
            if (camera && controls) {
                animateCameraPosition(0, 15, 0, 0, 0, 0);
            }
        };
        
        const setFrontView = () => {
            if (camera && controls) {
                animateCameraPosition(0, 2, 10, 0, 0, 0);
            }
        };
        
        const setSideView = () => {
            if (camera && controls) {
                animateCameraPosition(10, 2, 0, 0, 0, 0);
            }
        };
        
        // 相机动画函数 - 平滑过渡到新视角
        const animateCameraPosition = (camX, camY, camZ, targetX, targetY, targetZ) => {
            // 创建一个简单的动画效果
            const duration = 1000; // 动画持续时间（毫秒）
            
            // 记录初始位置
            const startCamPos = camera.position.clone();
            const startTarget = controls.target.clone();
            
            // 目标位置
            const endCamPos = new THREE.Vector3(camX, camY, camZ);
            const endTarget = new THREE.Vector3(targetX, targetY, targetZ);
            
            // 动画计时器
            let startTime = Date.now();
            
            // 动画函数
            function animateCamera() {
                const now = Date.now();
                const elapsed = now - startTime; // 经过的毫秒数
                const progress = Math.min(elapsed / duration, 1.0);
                
                // 使用缓动函数使动画更平滑
                const easeProgress = easeOutQuad(progress);
                
                // 插值计算新位置
                camera.position.lerpVectors(startCamPos, endCamPos, easeProgress);
                controls.target.lerpVectors(startTarget, endTarget, easeProgress);
                controls.update();
                
                if (progress < 1.0) {
                    requestAnimationFrame(animateCamera);
                }
            }
            
            // 缓动函数
            function easeOutQuad(t) {
                return t * (2 - t);
            }
            
            // 开始动画
            animateCamera();
        };

        // 设置全屏模式函数
        const toggleFullScreen = () => {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    console.error(`全屏错误: ${err.message}`);
                });
            } else {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                }
            }
        };

        onMounted(() => {
            // 创建场景
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x000000);

            // 添加相机
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 5, 10); // 设置相机位置（提高Y值可以让视角更高）

            // 渲染器
            renderer = new THREE.WebGLRenderer({ 
                antialias: true,
                alpha: true // 允许透明背景
            });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // 限制像素比以提高性能
            canvasContainer.value.appendChild(renderer.domElement);
            
            // 设置渲染器元素样式
            renderer.domElement.style.position = 'absolute';
            renderer.domElement.style.top = '0';
            renderer.domElement.style.left = '0';
            renderer.domElement.style.width = '100%';
            renderer.domElement.style.height = '100%';

            // 添加环境光
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
            scene.add(ambientLight);

            // 添加方向光
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(5, 5, 5);
            directionalLight.castShadow = true;
            scene.add(directionalLight);

            // 添加轨道控制器（允许鼠标拖拽旋转视角）
            controls = new OrbitControls(camera, renderer.domElement);
            controls.target.set(0, 0, 0); // 设置控制器的目标点（相机看向的位置）
            controls.enableDamping = true; // 添加阻尼效果
            controls.dampingFactor = 0.05;
            controls.screenSpacePanning = false;
            controls.minDistance = 1;  // 最小缩放距离
            controls.maxDistance = 100; // 最大缩放距离
            controls.minPolarAngle = Math.PI / 6; // 限制向上旋转角度（约30度）
            controls.maxPolarAngle = Math.PI / 1.5; // 限制上下旋转角度
            controls.enableZoom = true; // 启用缩放
            controls.enableRotate = true; // 启用旋转
            controls.enablePan = true; // 启用平移
            
            // 调整控制器速度
            controls.rotateSpeed = 1.0; // 旋转速度 (左键) - 默认值是1.0，值越大旋转速度越快
            controls.zoomSpeed = 1.2; // 缩放速度 (中键/滚轮) - 默认是1.0
            controls.panSpeed = 1.5; // 平移速度 (右键) - 默认是1.0，降低使右键拖动更精确
            
            // 配置鼠标按键功能 - 替代方案：右键也用于旋转
            // 如果您想让右键执行旋转操作，请取消下面的注释并注释掉当前的mouseButtons设置
            /*
            controls.mouseButtons = {
                LEFT: THREE.MOUSE.ROTATE,    // 左键旋转
                MIDDLE: THREE.MOUSE.DOLLY,   // 中键缩放
                RIGHT: THREE.MOUSE.ROTATE    // 右键也执行旋转
            };
            */
            
            // 配置鼠标按键功能，允许右键平移模型
            controls.mouseButtons = {
                LEFT: THREE.MOUSE.ROTATE,    // 左键旋转
                MIDDLE: THREE.MOUSE.DOLLY,   // 中键缩放
                RIGHT: THREE.MOUSE.PAN       // 右键平移
            };
            
            // 设置触摸操作
            controls.touches = { 
                ONE: THREE.TOUCH.ROTATE,
                TWO: THREE.TOUCH.DOLLY_PAN
            };

            // 更彻底地阻止浏览器默认的触摸行为
            const canvas = renderer.domElement;
            
            // 阻止浏览器默认的触摸行为
            canvas.addEventListener('touchstart', preventDefaultTouchActions, { passive: false, capture: true });
            canvas.addEventListener('touchmove', preventDefaultTouchActions, { passive: false, capture: true });
            canvas.addEventListener('touchend', preventDefaultTouchActions, { passive: false, capture: true });
            canvas.addEventListener('touchcancel', preventDefaultTouchActions, { passive: false, capture: true });
            canvas.addEventListener('wheel', preventDefaultTouchActions, { passive: false, capture: true });
            
            // 特别阻止右键菜单
            canvas.addEventListener('contextmenu', handleRightClick, { capture: true });
            document.addEventListener('contextmenu', handleRightClick, { capture: true }); // 全局阻止
            
            // 阻止鼠标右键的浏览器手势 - 增强版
            const handleMouseDown = (event) => {
                if (event.button === 2) { // 右键
                    event.preventDefault();
                    event.stopPropagation();
                    return false;
                }
            };
            
            canvas.addEventListener('mousedown', handleMouseDown, { passive: false, capture: true });
            document.addEventListener('mousedown', handleMouseDown, { passive: false, capture: true });
            
            // 阻止双击放大
            canvas.addEventListener('dblclick', preventDefaultTouchActions, { capture: true });
            
            // 添加pointer-events样式，确保触摸事件不会传递到下层元素
            canvas.style.touchAction = 'none';
            canvas.style.msTouchAction = 'none'; // 针对Edge/IE
            canvas.style.userSelect = 'none';
            canvas.style.webkitUserSelect = 'none';
            canvas.style.msUserSelect = 'none'; // 针对Edge/IE
            canvas.style.webkitTouchCallout = 'none';
            
            // Edge特有的手势取消属性
            document.documentElement.style.msContentZooming = 'none';
            document.documentElement.style.msOverflowStyle = 'none';
            
            // 禁用页面滚动
            document.body.style.overflow = 'hidden';
            document.documentElement.style.overflow = 'hidden';
            
            // 防止iOS和Edge的橡皮筋效果和手势
            const preventAllDefault = (e) => e.preventDefault();
            document.addEventListener('touchmove', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('gesturestart', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('gesturechange', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('gestureend', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('MSGestureStart', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('MSGestureChange', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('MSGestureEnd', preventAllDefault, { passive: false, capture: true });
            document.addEventListener('MSInertiaStart', preventAllDefault, { passive: false, capture: true });
            
            // 加载 GLTF 模型
            const loader = new GLTFLoader();
            loader.load(
                '/models/scene.gltf', // 确保路径正确
                (gltf) => {
                    const model = gltf.scene;
                    scene.add(model); // 添加模型到场景
                    
                    // 可选：自动调整相机位置以适应模型
                    const box = new THREE.Box3().setFromObject(model);
                    const center = box.getCenter(new THREE.Vector3());
                    const size = box.getSize(new THREE.Vector3());
                    
                    // 设置模型位置
                    model.position.x = -center.x;
                    model.position.y = -center.y;
                    model.position.z = -center.z;
                    
                    // 设置相机位置
                    const maxDim = Math.max(size.x, size.y, size.z);
                    const fov = camera.fov * (Math.PI / 180);
                    let cameraZ = Math.abs(maxDim / Math.sin(fov / 2));
                    camera.position.z = cameraZ * 1.5;
                    
                    // 更新控制器
                    controls.update();
                    
                    console.log('模型加载成功');
                },
                (xhr) => {
                    console.log(`加载进度: ${(xhr.loaded / xhr.total) * 100}%`);
                },
                (error) => {
                    console.error('加载错误', error);
                }
            );

            // 添加窗口大小变化监听
            window.addEventListener('resize', handleResize);
            
            // 在初始加载时，确保立即执行一次大小调整
            handleResize();

            // 添加键盘事件监听
            document.addEventListener('keydown', handleKeyDown);

            // 渲染循环
            function animate() {
                requestAnimationFrame(animate);
                controls.update(); // 更新控制器
                renderer.render(scene, camera);
            }

            animate();
            
            // 处理窗口可见性变化（标签页切换）
            document.addEventListener('visibilitychange', handleVisibilityChange);
        });
        
        // 组件卸载前清理资源
        onBeforeUnmount(() => {
            window.removeEventListener('resize', handleResize);
            
            // 创建一个通用的移除监听器函数
            const removeEventListeners = (element, eventName, handler) => {
                if (element) {
                    element.removeEventListener(eventName, handler, { capture: true });
                    element.removeEventListener(eventName, handler, { passive: false, capture: true });
                }
            };
            
            // 移除触摸事件监听器
            if (renderer && renderer.domElement) {
                const canvas = renderer.domElement;
                
                // 移除canvas上的事件监听器
                removeEventListeners(canvas, 'touchstart', preventDefaultTouchActions);
                removeEventListeners(canvas, 'touchmove', preventDefaultTouchActions);
                removeEventListeners(canvas, 'touchend', preventDefaultTouchActions);
                removeEventListeners(canvas, 'touchcancel', preventDefaultTouchActions);
                removeEventListeners(canvas, 'wheel', preventDefaultTouchActions);
                removeEventListeners(canvas, 'contextmenu', handleRightClick);
                removeEventListeners(canvas, 'dblclick', preventDefaultTouchActions);
                removeEventListeners(canvas, 'mousedown', preventDefaultTouchActions);
            }
            
            // 移除文档上的事件监听器
            const preventAllDefault = (e) => e.preventDefault();
            removeEventListeners(document, 'contextmenu', handleRightClick);
            removeEventListeners(document, 'mousedown', preventDefaultTouchActions);
            removeEventListeners(document, 'touchmove', preventAllDefault);
            removeEventListeners(document, 'gesturestart', preventAllDefault);
            removeEventListeners(document, 'gesturechange', preventAllDefault);
            removeEventListeners(document, 'gestureend', preventAllDefault);
            removeEventListeners(document, 'MSGestureStart', preventAllDefault);
            removeEventListeners(document, 'MSGestureChange', preventAllDefault);
            removeEventListeners(document, 'MSGestureEnd', preventAllDefault);
            removeEventListeners(document, 'MSInertiaStart', preventAllDefault);
            
            // 恢复页面滚动
            document.body.style.overflow = '';
            document.documentElement.style.overflow = '';
            
            if (renderer) {
                renderer.dispose();
            }
            if (controls) {
                controls.dispose();
            }
            
            // 移除键盘事件监听
            document.removeEventListener('keydown', handleKeyDown);
            
            // 移除额外添加的事件监听器
            document.removeEventListener('visibilitychange', handleVisibilityChange);
        });

        return {
            canvasContainer,
            moveUp,
            moveDown,
            setTopView,
            setFrontView,
            setSideView,
            toggleFullScreen
        };
    },
};
</script>

<style>
/* 全局禁用页面上的浏览器默认触摸行为 */
html, body {
    margin: 0;
    padding: 0;
    overscroll-behavior-x: none;
    overscroll-behavior-y: none;
    touch-action: none;
    -ms-touch-action: none; /* 针对Edge浏览器 */
    -webkit-overflow-scrolling: auto;
    position: fixed;
    width: 100%;
    height: 100%;
    overflow: hidden;
    
    /* Edge特有的属性 */
    -ms-content-zooming: none;
    -ms-overflow-style: none;
}

/* 禁用所有可能的手势相关效果 */
* {
    -webkit-tap-highlight-color: transparent;
    -webkit-touch-callout: none;
    -ms-touch-select: none; /* Edge特有 */
    touch-action: none;
    -ms-touch-action: none;
    box-sizing: border-box;
}

#app {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
}
</style>