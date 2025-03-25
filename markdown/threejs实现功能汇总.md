#### threejs实现功能汇总

1. 给模型添加轮廓线

   ```ts
   //  添加轮廓线
   export const showContourlines = (mesh: THREE.Mesh) => {
     if (!mesh) return
     mesh.traverse((child: THREE.Object3D<THREE.Object3DEventMap>) => {
       if (child instanceof THREE.Mesh) {
         const edges = new THREE.EdgesGeometry(child.geometry, 80)
         const lineMaterial = new THREE.LineBasicMaterial({ color: 0x000000 })
         const wireframe = new THREE.LineSegments(edges, lineMaterial)
         wireframe.name = "wireframe"
         child.add(wireframe)
       }
     })
   }
   ```

2. 清理场景内的所有模型

   ```ts
   export const clearScene = (scene: THREE.Scene) => {
     // 遍历场景中的所有Mesh对象
     while (scene.children.length > 0) {
       // 获取第一个子对象
       const object = scene.children[0] as THREE.Mesh
       // scene.background = null
       // geometry（几何体）或material（材质）可以在3D物体之间共享,所以THREE不会主动移除
       if (object.isMesh) {
         // 如果Mesh使用了几何体，释放几何体
         if (object.geometry) {
           object.geometry.dispose()
         }
         // 如果Mesh使用了材料，释放材料
         if (object.material) {
           if (Array.isArray(object.material)) {
             // 对于数组材料，遍历并释放每个材料
             object.material.forEach((material: any) => material.dispose())
           } else {
             // 对于单个材料，直接释放
             object.material.dispose()
           }
         }
       }
       // 从场景中移除对象
       scene.remove(object)
     }
     // 可选：如果有需要，也可以清理其他资源，如纹理
   }
   ```

3. 切换视图视角

   ```ts
   //  变换视角
   export const changeFace = (i: number) => {
     const { camera } = initialThree.getInitialData()
     // console.log("TCL: changeFace -> i", i)
     //   0     1     4     5     3     2
     // +蓝z   -蓝z  +红x  -红x  -绿y   +绿y
     const distanceToOrigin = camera.position.distanceTo(new THREE.Vector3(0, 0, 0))
     const positionArr = [
       new THREE.Vector3(0, 0, distanceToOrigin), // 正上方
       new THREE.Vector3(0, 0, -distanceToOrigin), // 正下方
       new THREE.Vector3(0, -distanceToOrigin, 0), // 正左方
       new THREE.Vector3(0, distanceToOrigin, 0), // 正右方
       new THREE.Vector3(distanceToOrigin, 0, 0), // 正前方
       new THREE.Vector3(-distanceToOrigin, 0, 0), // 正后方
       new THREE.Vector3(-distanceToOrigin, distanceToOrigin, distanceToOrigin), // zuo后方
     ]
     camera.position.copy(positionArr[i] || positionArr[0])
   }
   ```

4. 创建包围盒,并显示size

   ```ts
   // 创建文本精灵(文字)
   const createTextSprite = (text: string, scale: number) => {
     const canvas = document.createElement("canvas")
     const context = canvas.getContext("2d")
     if (!context) return null
     // 设置较大的分辨率和字体
     const fontSize = 20 // 增大字体大小
     const padding = 10 // 内边距
     context.font = `Bold ${fontSize}px Arial`
     context.fillStyle = "#e53333"
     // 计算文本宽度，调整canvas大小
     const metrics = context.measureText(text)
     const textWidth = metrics.width
     canvas.width = textWidth + padding * 2 // 加一些填充
     canvas.height = fontSize + padding * 2 // 固定高度
     // 重新绘制文本到调整过大小的canvas上
     context.font = `Bold ${fontSize}px Arial`
     context.fillStyle = "#e53333"
     // canvas.zIndex = 9999999
     context.fillText(text, padding, fontSize + padding)
   
     const texture = new THREE.CanvasTexture(canvas)
     texture.needsUpdate = true
   
     const spriteMaterial = new THREE.SpriteMaterial({ map: texture })
     const sprite = new THREE.Sprite(spriteMaterial)
     sprite.renderOrder = 999 // 确保精灵渲染在其他对象之上  ??  不需要
   
     // 调整比例以适应场景，使用固定的比例
     // const scale = 10 // 固定大小的比例
     sprite.scale.set(scale, scale * (canvas.height / canvas.width), 1)
   
     return sprite
   }
   
   const addSizeLabels = (mesh: THREE.Mesh) => {
     // const size = new THREE.Vector3()
     // box.getSize(size)
     const boxHelper = new THREE.BoxHelper(mesh, 0x3ff94e)
     const box = new THREE.Box3().setFromObject(mesh)
     // scene.add(boxHelper)
     const labelArr: any = [boxHelper]
     const size = box.getSize(new THREE.Vector3())
     //  此处scale 用于获得相机距离模型的距离  从而计算 文本放大比例
     const d = Math.sqrt(size.x * size.x + size.y * size.y)
     const scale = Number((d / 4).toFixed(2))
     const positions = [
       {
         text: `长: ${size.x.toFixed(2)}`,
         position: new THREE.Vector3((box.min.x + box.max.x) / 2, box.min.y, box.min.z),
       },
       {
         text: `宽: ${size.y.toFixed(2)}`,
         position: new THREE.Vector3(box.min.x, (box.min.y + box.max.y) / 2, box.min.z),
       },
       {
         text: `高: ${size.z.toFixed(2)}`,
         position: new THREE.Vector3(box.min.x, box.min.y, (box.min.z + box.max.z) / 2),
       },
     ]
     positions.forEach((dimension) => {
       const sprite = createTextSprite(dimension.text, scale)
       if (!sprite) return labelArr
       sprite.position.copy(dimension.position)
       sprite.material.depthTest = false // 确保文本不被遮挡
       // scene.add(sprite) // 添加到场景中
       labelArr.push(sprite)
     })
     // 调整每个标签的位置，使其位于对应的线条中间
     positions[0].position.set((box.min.x + box.max.x) / 2, box.min.y - 0.1, box.min.z - 0.1) // 长：底部中间
     positions[1].position.set(box.min.x - 0.1, (box.min.y + box.max.y) / 2, box.min.z - 0.1) // 宽：左侧中间
     positions[2].position.set(box.min.x - 0.1, box.min.y - 0.1, (box.min.z + box.max.z) / 2) // 高：前面中间
   
     positions.forEach((dimension) => {
       const sprite = createTextSprite(dimension.text, scale)
       if (!sprite) return labelArr
   
       sprite.position.copy(dimension.position)
       // sprite.material.depthTest = false; // 确保文本不被遮挡
       sprite.position.z += 0.1 // 防止与其他对象重叠
       // scene.add(sprite) // 添加到场景中
       labelArr.push(sprite)
     })
     return labelArr
   }
   ```

5. 模型切换透明

   ```ts
   export const toggleTransparent = (mesh: THREE.Mesh | any) => {
     if (!mesh || !mesh.material) {
       console.warn("Invalid mesh or material provided.")
       return
     }
     // 反转透明状态标志
     const isTransparent = mesh.userData?.isTransparent || false
     // 更新透明属性
     const newOpacity = isTransparent ? 1 : 0.5
     const newTransparent = !isTransparent
     // 如果 mesh 是一个容器，遍历其子元素
     mesh.traverse?.((child: any) => {
       if (child.isMesh && child.material) {
         child.material.opacity = newOpacity
         child.material.transparent = newTransparent
         child.material.side = THREE.FrontSide // 确保前面可见
       }
     })
     // 如果 mesh 是单一对象
     if (mesh.material) {
       mesh.material.opacity = newOpacity
       mesh.material.transparent = newTransparent
       mesh.material.side = THREE.FrontSide // 确保前面可见
     }
     // 更新用户数据中的透明状态标志
     mesh.userData.isTransparent = newTransparent
   }
   ```

6. 鼠标点击,选中获取当前模型

   ```ts
   const getMesh = (event: MouseEvent) => {
         const { raycaster, camera, mouse, renderer } = initialThree.getInitialData()
         mouse.x =
           ((event.clientX - renderer.domElement.getBoundingClientRect().left) / renderer.domElement.clientWidth) * 2 - 1
         mouse.y =
           -((event.clientY - renderer.domElement.getBoundingClientRect().top) / renderer.domElement.clientHeight) * 2 + 1
   
         raycaster.setFromCamera(mouse, camera)
         const intersects = raycaster.intersectObjects(useThreeStore.getState().meshArr, true)
         for (let i = 0; i < intersects.length; i++) {
           if (intersects[i].object.isMesh) {
             // 当前面的是其他材质  需要一直往后取 直到找到mesh
             setCurrentMesh(intersects[i].object)  // 得到mesh
             break
           }
         }
       }
   //  添加监听事件
   window.addEventListener("click", getMesh)
   ```

7. 根据模型自动适配渲染截图

   ```ts
   // 先比较获取模型面积最大的面, 截图后再切换成左上视角,保存为初始视图
   renderer.render(scene, camera)
   const imageUrl = renderer.domElement.toDataURL("image/jpeg")
   

8. 

9. 