<template>
  <div style="margin: 20px">
    <a-descriptions title="作者信息" bordered column="3">
      <a-descriptions-item label="姓名">李烁程</a-descriptions-item>
      <a-descriptions-item label="学院">计算机科学与技术学院</a-descriptions-item>
      <a-descriptions-item label="专业">计算机科学与技术（嵌入式系统方向）</a-descriptions-item>
      <a-descriptions-item label="学号">19169100001</a-descriptions-item>
      <a-descriptions-item></a-descriptions-item>
      <a-descriptions-item></a-descriptions-item>
      <a-descriptions-item label="指导教师姓名">王凯东</a-descriptions-item>
      <a-descriptions-item label="职称">副教授</a-descriptions-item>
      <a-descriptions-item label="指导教师单位">计算机科学与技术学院</a-descriptions-item>
    </a-descriptions>

    <br/>

    <a-descriptions title="课题信息" bordered layout="vertical" :column="3">
      <a-descriptions-item label="名称" :span="3">基于微服务的生产过程中质量品控系统的设计与实现</a-descriptions-item>
      <a-descriptions-item label="英文名称" :span="3">Design and implementation of quality control system in production process based on microservices</a-descriptions-item>
      <a-descriptions-item label="课题来源" :span="2">自设科研项目</a-descriptions-item>
      <a-descriptions-item label="课题性质">软件</a-descriptions-item>
      <a-descriptions-item label="课题难度">中</a-descriptions-item>
      <a-descriptions-item label="是否重点扶持项目">否</a-descriptions-item>
      <a-descriptions-item label="审题学院">计算机科学与技术学院</a-descriptions-item>
      <a-descriptions-item label="邮箱"></a-descriptions-item>
      <a-descriptions-item label="手机号码"></a-descriptions-item>
      <a-descriptions-item label="办公区域"></a-descriptions-item>
      <a-descriptions-item label="课题简介">
        维基上对微服务定义为：一种软件开发技术- 面向服务的体系结构（SOA）架构样式的一种变体，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务间采用轻量级的通信机制互相沟通（通常是基于HTTP的RESTful API）。每个服务都围绕着具体业务进行构建，并且能够独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据上下文，选择合适的语言、工具对其进行构建。
        <br/>
        生产过程控制(production process control)为确保生产过程处于受控状态，对直接或间接影响产品质量的生产、安装和服务过程所采取的作业技术和生产过程的分析，诊断和监控。
        <br/>
        统计过程控制（Statistical Process Control）是一种借助数理统计方法的过程控制工具。它对生产过程进行分析评价，根据反馈信息及时发现系统性因素出现的征兆，并采取措施消除其影响，使过程维持在仅受随机性因素影响的受控状态，以达到控制质量的目的。统计过程控制一些控制图算法有相关文献资料由老师提供。
        <br/>
        本课题要求使用Java的低代码开发平台JeecgBoot，它是一款基于BPM的低代码平台！前后端分离架构 SpringBoot 2.x，SpringCloud，Ant Design&Vue，Mybatis-plus，Shiro，JWT，支持微服务。开发出统计过程相关控制图模块，具体功能模块参考如下图示。
        <br/>
      </a-descriptions-item>
    </a-descriptions>

    <div style="margin: 20px">
      <img src="/resource/img/spc.png" title="spc.png" width="2000">
    </div>


    <br/>
  </div>
</template>

<script>
export default {
  name: "about"
}
</script>

<style scoped>

</style>
