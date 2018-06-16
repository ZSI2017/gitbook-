## 1.9 其他行为 ##
### 1.9.1 分享 ###

### 1.9 其他行为 ###
#### 1.9.1 分享 ####

#### 1.9.2 点赞&取消点赞 ####
**通过customize-item 类，绑定了对应的trackData,走统一的自定义上报逻辑**

点赞&取消点赞, 上报内容的字段中， `category`字段对应“点赞”或者“取消点赞”。






```
   <a class="{{=moreClass}} customize-item" {{<{__init__: [like__init], track: ['点赞', trackName, cate, 1, cate]}}}>

```




#### 1.9.3 负反馈（分为广告类、非广告类） ####  
**函数：** `track__dislikeFeed`
 ```
 function track__dislikeFeed(args) {
     var type = args.type,
         trackData = {}; //自定义事件数据

     if (type == 'ad') { //广告数据
         _track__adDislikeFeed(args, trackData);
     } else { //非广告数据
         _track__itemDislikeFeed(args, trackData);
     }

     if (args.from == 3) {
         return;
     }

     O2oStats__trackEvent({
         cate: trackData.category,
         action: trackData.action,
         name: trackData.name,
         ext: JSON.stringify(trackData.value),
     });
 }
 ```

**场景：**
 **三种相关动作（代码中`from`字段区分）：**
  - 点击负反馈按钮“x”（调起负反馈弹窗） --1
  - 点击“确定”/“不感兴趣” 按钮（`.dislike__btn`类名标识，弹窗消失） -- 2
  - 点击弹窗背景（弹窗消失） --3

 *点击不同理由，没有触发事件上报。*

**每种动作分为 广告类 和 非广告类，广告类负反馈看下方广告类事件。**

**备注：**
   所有负反馈相关的动作，都会触发`track__dislikeFeed`函数，里面进行广告和非广告条目区分。
   这里主要看`_track__itemDislikeFeed`函数，非广告相关的负反馈上报。

   里面上报字段中`action`,区分“精品库/非精品库”,
   ```
     if (itemType && itemType.indexOf('cms') > -1) {
         isCms = true;

         trackData = Object__extend(trackData, {
             action: '精品库',
         });

         //精品库卡片取标题为反馈的类别
         $card = Dom__closest(target, '.card');

         if ($card) {
             cardTitle = Dom__data($card) && Dom__data($card).data;

             trackData.value = Object__extend(trackData.value, {
                 feed_cate: cardTitle || '',
             });
         }
     } else {
         isCms = false;

         trackData = Object__extend(trackData, {
             action: '非精品库',
         });
     }

   ```

  `category`字段，区分不同场景下上报,from 字段判断参考;

  ```
  switch (from) {
      case 1:
          trackData = Object__extend(trackData, {
              category: '非广告负反馈按钮点击',
          });
          break;
      case 2:
          if (reasons && reasons.length) {
              trackData = Object__extend(trackData, {
                  category: '非广告负反馈原因提交点击',
              });
              trackData.value = Object__extend(trackData.value, {
                  feed_reason: reasons,
              });
          } else {
              trackData = Object__extend(trackData, {
                  category: '非广告负反馈无原因提交点击',
              });
          }

          //reachType为20的负反馈上报
          reachData.ext = reachData.ext || {};

          Object__extend(reachData.ext, {
              dislike_reason: reasons || [],
              action: isCms ? '精品库' : '非精品库',
          });
          // 点击了 “确定”/“不感兴趣” ，上报‘reachNoInterest’
          _track__send('reachNoInterest', reachData);
          break;
      default:
          break;
  }

  ```

####1.9.4 Cms卡片
 **根据接口下发的每个条目中，对应的 type字段中，是否包含“cms”,判断是否为精品库**
 **场景：**
   - 非广告负反馈，`actions`字段，区分精品库和非精品库
   - 条目触达点击事件，对精品库条目，单独上报一次；
     ```
     //cms上报
     if (data.itemType && data.itemType.toLowerCase().indexOf('cms') > -1) {
         O2oStats__trackEvent({
             cate: 'cms卡片',
             action: '点击',
             name: category || '未知',
         });
     }
     ```



####1.9.5 落地页（视频条目点击来源进入落地页）


####1.9.6 更多按钮点击
- 新闻类卡片

  **场景：**

  **备注：**
     ```
      {% raw %}
      <a href="{{=data.buttons[0].url}}" class="card__more customize-item" {{<{track: data.mtype=='News'?['新闻卡更多', '点击', data.title, 1, pathName]: [data.title, '点击', data.buttons[0].title, 1, pathName]}}}>更多</a>
      {% endraw %}
     ```  
    在tpl 模板中绑定 `trackData`数据， 走`customize-item`类识别的自定义统计逻辑上报。
- 其他类卡片
   **场景：** ...

####1.9.7 自有新闻页(news-cp项目)  


####1.9.8 Push调起
  **场景：** push 调起，客户端回调函数中，自定义事件上报。

  **备注：**
    ```  
        O2oStats__trackEvent({
         cate: 'PUSH',
         action: '自定义推送',
     });

    ```


###1.10 进入短视频
#### 进入短视频 ####
  **自定义事件，上报进入短视频,在进入短视频后的客户端方法回调中进行上报。**
   ```
     O2oStats__trackEvent({
         cate: '进入短视频',
         action: '点击短视频icon',
         name: cate.name,
         eid: eid,
         path: cate.name,
     });
   ```

###1.11 退出短视频
  **自定义事件，不同方式退出，调用客户端方法，回调函数中上报不同的action**

  ```
    switch (from) {
        case 0:
            action = '点击back';
            break;
        case 1:
            action = '点击主页';
            break;
        case 2:
            action = '点击资讯icon';
            break;
        default:
            action = '退出短视频';
    }

    O2oStats__trackEvent({
        cate: '退出短视频',
        action: action,
        name: cate.name,
        eid: eid,
        path: cate.name,
    });

  ```

  **场景：**
  - 点击底部资讯icon
  - 点击back
  - 点击主页
  - 退出短视频

    - 非1. 2. 3. 情况，默认4. action
