# 视频图像流处理

## 背景去除
      背景减除 Background subtraction (BS)
      背景减除在很多基础应用中占据很重要的角色。
      列如顾客统计，使用一个静态的摄像头来记录进入和离开房间的人数，
      或者交通摄像头，需要提取交通工具的信息等。
      我们需要把单独的人或者交通工具从背景中提取出来。
      技术上说，我们需要从静止的背景中提取移动的前景.

      提供一个无干扰的背景图像
      实时计算当前图像和 背景图像的差异（图像做差）  阈值二值化 得到 多出来的物体(mask)   再区域分割

      BackgroundSubtractorMOG2 是以高斯混合模型为基础的背景/前景分割算法。
      它是以2004年和2006年Z.Zivkovic的两篇文章为基础。
      这个算法的一个特点是它为每个像素选择一个合适的高斯分布。
      这个方法有一个参数detectShadows，默认为True，他会检测并将影子标记出来，
      但是这样做会降低处理速度。影子会被标记成灰色。


       背景与前景都是相对的概念，以高速公路为例：有时我们对高速公路上来来往往的汽车感兴趣，
      这时汽车是前景，而路面以及周围的环境是背景；有时我们仅仅对闯入高速公路的行人感兴趣，
      这时闯入者是前景，而包括汽车之类的其他东西又成了背景。背景剪除是使用非常广泛的摄像头视频中探测移动的物体。
      这种在不同的帧中检测移动的物体叫做背景模型，其实背景剪除也是前景检测。



      一个强劲的背景剪除算法应当能够解决光强的变化，杂波的重复性运动，和长期场景的变动。
      下面的分析中会是用函数V(x,y,t)表示视频流，t是时间，x和y代表像素点位置。
      例如，V(1,2,3)是在t=3时刻，像素点(1,2)的光强。下面介绍几种背景剪除的方法。


      【1】 利用帧的不同（临帧差）
        该方法假定是前景是会动的，而背景是不会动的，而两个帧的不同可以用下面的公式：
          D(t+1) = I(x,y,t+1) - I(x,y,t)
        用它来表示同个位置前后不同时刻的光强只差。
        只要把那些D是0的点取出来，就是我们的前景，同时也完成了背景剪除。
        当然，这里的可以稍作改进，不一定说背景是一定不会动的。
        可以用一个阀值来限定。看下面的公式：
            I(x,y,t+1) - I(x,y,t) > 阈值
       通过Th这个阀值来进行限定，把大于Th的点给去掉，留下的就是我们想要的前景。

          #define threshold_diff 10 临帧差阈值
          // 可使用 矩阵相减 
          subtract(gray1, gray2, sub);  

        for (int i = 0;i<bac.rows; i++)  
            for (int j = 0;j<bac.cols; j++)  
          if (abs(bac.at<unsigned char>(i, j)) >= threshold_diff)
                    //这里模板参数一定要用 unsigned char 8位(灰度图)，否则就一直报错  
              bac.at<unsigned char>(i, j) = 255;  
          else bac.at<unsigned char>(i, j) = 0;



      【2】 均值滤波（Mean filter） 当前帧 和 过去帧均值  做差

            M(x,y,t) = 1/N SUM(I(x,y,(1..t)))
            I(x,y,t+1) - M(x,y,t) > 阈值        为前景

        不是通过当前帧和上一帧的差别，而是当前帧和过去一段时间的平均差别来进行比较，
        同时通过阀值Th来进行控制。当大于阀值的点去掉，留下的就是我们要的前景了。

      【3】 使用高斯均值  + 马尔科夫过程
      // 初始化：
           均值 u0 = I0 均值为第一帧图像
           方差 c0 = (默认值)
      // 迭代：
             ut = m * It + (1-m) * ut-1       马尔科夫过程
             d  = |It - ut|                   计算差值
             ct = m * d^2  + (1-m) * ct-1^2   更新 方差

      判断矩阵 d/ct > 阈值    为前景