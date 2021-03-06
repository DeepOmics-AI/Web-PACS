# Web-PACS

基于开源库实现云PACS的方案

越来越多的项目希望使用web来访问PACS，把数据和服务器托管在云端，实现影像云或者云PACS。折腾了几年之后，总结一下调查过的开源方案，列举一下相应的有优缺点和适应范围。不一定使所有可能的方案，不过肯定是相对靠谱的方案。

架构和开源库

从最终客户角度，可以把系统结构分成两个流派：1）浏览器端完成DICOM数据解析和渲染，2）服务器端完成解析和渲染，浏览器只负责简单的显示和交互。

前端渲染

通过html和javascript实现前端渲染，包括三维体绘制都是可以实现的，并且渲染的性能也不影响用户体验。但是，浏览器都有一个重要的限制，内存使用量。一般一个网页可以占用的内存大约1-4G，根据操作系统和硬件平台有关。这是一个浏览器通用的安全保护。这样，就会造成没有办法在浏览器端处理大型的体数据。同时，浏览器可以使用的本地缓存也有比较多的限制。即使是二维图像操作，如何制定合理的缓存策略也是一个需要小心的地方。

比较完善的开源项目有这几个：

cornerstone
https://github.com/chafey/cornerstone

这是一个纯前端js实现的DICOM二维处理。作者Chris Hafey是行业资深人士。完整的演示代码可以在https://github.com/chafey/cornerstoneDemo找到。这个演示基本上就是下载一个json表示study信息，然后直接用http获取dicom文件到前端处理。后台实际上没有真正意义的服务器。

这个库的好处是容易理解，服务器端简单。常用功能比较完善，bug比较少。适用于一些对dicom操作要求不高，只需要处理DR或者低端CT/MR图像的地方，比较容易集成。

新版本 https://cornerstonejs.org/ 架构更完善“>https://github.com/cornerstonejs/cornerstone_

oviyam
https://sourceforge.net/p/dcm4che/svn/HEAD/tree/oviyam/oviyam/

这个项目和DCM4CHE捆绑比较紧。如果PACS后端采用DCM4CHE，这是一个可行的选择。功能和界面相对更完善，但是，实际测试中发现的小bug比较多。需要对DICOM有一定了解才能比较好的定制和维护。

The X Toolkit
https://github.com/xtk/X

这个项目是波士顿儿童医院和哈佛医学院合作的开源项目。提供里比较完整的三维渲染功能，甚至包括一些很高端的脑部核磁图像处理。所有功能都基于前端js实现。主要的问题就是前面提到的数据体大小的问题。一旦超出内存限制，就会出错。

解决内存限制还有一个方法就是自行定制浏览器，这个超出了web版的使用场景，就不在这深入讨论了。

vtk-js
Visualization Toolkit for the Web https://kitware.github.io/vtk-js/

VTK的子集前端实现，对于熟悉VTK的同学，可以考虑。

服务器端渲染

考虑到高端CT/MR经常出现几千张图的序列，可能会遇到几个问题：1）浏览器可能没有办法装入整个序列，2）在现实带宽下，下载几百兆数据需要的用户等待时间过长，3）数据下载到前端加大了数据泄露的可能。所以，对于云端的PACS，服务器端渲染是一个比较好的选择。

服务器端渲染需要解决的核心问题是负载均衡和资源调度。再强悍的机器可以同时支持的渲染数量也会受到显卡、内存以及CPU的限制。不同客户的数据需要避免相互干扰。另外，如何保证服务器多活也需要考虑。

可以借助的开源方案有两个：

paraviewweb
https://github.com/Kitware/paraviewweb

这是最早从paraview里面出去出来的web框架。在vtk和paraview中都包括这部分代码。它实现了完整的进程和负载均衡管理，以及VTK接口到前端的映射。他自己的负载均衡就直接可以控制多台服务器。如果不在乎重度依赖VTK的框架，是一个不错的选择。

wt
https://www.webtoolkit.eu/wt

webtoolkt主要用途是C++直接产生操作WEB页面。这个框架通过一定虚拟化把html的页面元素映射成自己界面元素，提供相应的交互。这样，后端的渲染窗口直接生成图像返回前端就可以实现web展示。

真正的渲染可以用任何vtk或者其他opengl渲染库来实现。看开发团队的技术积累。

这个框架实现了单机的负载均衡，可以为不同客户端请求自动启动不同渲染进程。配合其他7层负载均衡软件，比如haproxy，可以很好的实现多机负载均衡。

服务器端渲染如果需要部署在公有云上，还需要考虑同时支持GPU和CPU渲染。目前只有一小部分公有云提供带GPU渲染的服务器，比如，AWS。大多数公有云还没有提供。同时，一些公有云提供用于深度学习的GPU虚机，用来做PACS渲染的可能性价比需要评估。

