+++
[banner]
  image = "image/banner.jpg"

  [[banner.button]]
      url = "/posts"
      text = "My posts"
      type = "primary"

  [[banner.button]]
      url = "/about"
      text = "About me"
      type = "primary"

#Details for the box below the banner
[services]
  title = "A thousand foe won't bent my will."
  text = ""



[feature_icons]
  #These feature icons look best if there's an even number of them.
  enable = true

  #Accent is a colour defined in the CSS file. Choose between 1 and 5
  [[feature_icons.tile]]
    icon = "fa-database"
    icon_pack = "fas"
    accent = "1"
    title = "数据分析"
    text = "异构环境下的高效数据存储机制针对当前基于Hadoop的海量网络数据处理平台中数据存储问题"

  [[feature_icons.tile]]
    icon = "fa-server"
    icon_pack = "fas"
    accent = "4"
    title = "太阳能互联网光束无人机"
    text = "Facebook自己的连接实验室开发了一种太阳能无人机，其翼展与波音747一样大，有一天，Facebook希望高能效的无人机将在60英里半径范围内飞行，同时在任何需要的地方提供互联网接入"

  [[feature_icons.tile]]
    icon = "fa-tv"
    icon_pack = "fas"
    accent = "5"
    title = "Project Loon浮动基于气球的互联网接入"
    text = "Google已推出Project Loon，这是一项依赖气球的类似计划。高空互联网配备的气球旨在传播非洲和东南亚农村地区的连通性，以及其他任何可以进入的地方。"

  [[feature_icons.tile]]
    icon = "fa-biohazard"
    icon_pack = "fas"
    accent = "3"
    title = "Loved"
    text = "热爱我所热爱"

[feature_images]
#These feature images look best if there's an even number of them.
  enable = true

  [[feature_images.tile]]
    image = "/image/xml/run.jpg"
    text = "Use of TinyXML"
    url = "/posts/tinyxml2/"
    button_text = "Learn more"

  [[feature_images.tile]]
    image = "/image/webhook/nodejs.jpg"

    text = "Webhook with nginx"
    url = "/posts/auto_deploy/"
    button_text="Learn more"

[CTA]
  heading = "Have a nice day!"
  message = "I'd love to hear from you,please feel free to contact me."
+++
