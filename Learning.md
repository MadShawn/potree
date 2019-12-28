## 点云octree文件的加载流程  
以下是伪代码  
``` javascript
viewer = new Potree.Viewer({
    requestAnimationFrame(this.loop.bind(this));

    this.loop({
        requestAnimationFrame(this.loop.bind(this)); // 连续调用loop
        
        Potree.updatePointClouds(this.scene.pointclouds)
            => Potree.updateVisibility(this.scene.pointclouds)
            => for (potree of this.scene.pointclouds) {
                pcogNodes = get pcogNodes from potree.root;

                unloadedGeometry = filter unloaded visable pcogNodes

                for (pcogNode of unloadedGeometry) {
                    pcogNode.loadPoints()
                }
            }
        });
    })
})

Potree.loadPointCloud(cloud.js) 
    => POCLoader.load(cloud.js)
                .then({
                    pcog = new PointCloudOctreeGeometry();
                    pcogNode = new PointCloudOctreeGeometryNode(pcog);

                    pcogNode.load() 
                         => pcogNode.loadHierachy(r.hrc)
                                   .then({
                                          build tree relationship of pcogNode;
                                          pcogNode.loadPoints()
                                               => pcog.loader.load(pcogNode)
                                               => BinaryLoader.load(r.bin)
                                                              .then({
                                                                get worker of BinaryDecoderWorker;

                                                                worker.onmessage({
                                                                    pcogNode.loaded = true;
                                                                });

                                                                worker.postmessage() =====> worker do the render
                                                              })
                                    })

                    pcog.setRoot(pcogNode);

                    return pcog;
                })
                .then({
                    pcotree = new PointCloudOctree(pcog);
                    viewer.scene.addPointCloud(pcotree);
                })
```
