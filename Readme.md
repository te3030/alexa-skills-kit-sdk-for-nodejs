
# [源地址](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs)


# 基于Node.js的Alexa Skills Kit SDK
<!-- TOC -->

- [基于Node.js的Alexa Skills Kit SDK](#基于Node.js的Alexa Skills Kit SDK)
    - [概述](#概述)
    - [安装指南](#安装指南)
    - [入门篇：编写Hello World Skill](#入门篇：编写Hello World Skill)
        - [基本项目结构](#基本项目结构)
        - [Set Entry Point](#set-entry-point)
        - [Implement Handler Functions](#implement-handler-functions)
    - [Response vs ResponseBuilder](#response-vs-responsebuilder)
        - [Tips](#tips)
    - [Standard Request and Response](#standard-request-and-response)
    - [Interfaces](#interfaces)
        - [AudioPlayer Interface](#audioplayer-interface)
        - [Dialog Interface](#dialog-interface)
            - [Delegate Directive](#delegate-directive)
            - [Elicit Slot Directive](#elicit-slot-directive)
            - [Confirm Slot Directive](#confirm-slot-directive)
            - [Confirm Intent Directive](#confirm-intent-directive)
        - [Display Interface](#display-interface)
        - [Playback Controller Interface](#playback-controller-interface)
        - [VideoApp Interface](#videoapp-interface)
        - [Skill and List Events](#skill-and-list-events)
    - [Services](#services)
        - [Device Address Service](#device-address-service)
        - [List Management Service](#list-management-service)
        - [Directive Service](#directive-service)
    - [Extend Features](#extend-features)
        - [Skill State Management](#skill-state-management)
        - [Persisting Skill Attributes through DynamoDB](#persisting-skill-attributes-through-dynamodb)
        - [Adding Multi-Language Support for Skill](#adding-multi-language-support-for-skill)
        - [Device ID Support](#device-id-support)
        - [Speechcons (Interjections)](#speechcons-interjections)
    - [Setting up your development environment](#setting-up-your-development-environment)

<!-- /TOC -->

## 概述

Alexa SDK团队很荣幸地推出新的 **Alexa Node.js SDK** ，这是developers为developers开发的开源Alexa Skill的开发工具包。

使用 Alexa Skill Kit 工具包，Node.js和AWS Lambda已经成为我们今天看到的 Skill 最流行的方式之一。 Node.js的事件驱动的非阻塞I / O模型非常适合Alexa Skill，Node.js是世界上最大的开源库生态系统之一。 此外，AWS Lambda每月免费提供第一百万个电话，这对于大多数Alexa Skills开发人员来说已经足够了。 此外，使用AWS Lambda时，您不需要管理任何SSL证书，因为Alexa Skills Kit是可信的触发器。

使用AWS Lambda，Node.js和Alexa skill构建Alexa Skills Kit是一个简单的过程。 但是，你实际需要编写的代码量却没有那么多。Alexa SDK团队现在已经构建了一个专门用于Node.js的Alexa Skills Kit SDK，它将帮助您避免常见的困惑，并专注于 Skill 的逻辑而不是样板代码。

使用新的alexa-sdk，我们的目标是帮助您更快地去构建skills，而让你避免不必要的复杂性。 今天，我们将推出具有以下功能的SDK：

- NPM作为包管理器，允许简单部署到任何Node.js环境
- 使用内置事件建立Alexa响应的能力
- 新会话和未处理事件作为Helper事件，可以充当'catch-all' 事件
- Helper函数用于构建基于state-machine（状态机）的Intent处理
- 可以根据skill去处理当前状态定义下不同的事件程序
- 简单使用DynamoDB的属性启用持久性的配置
- 所有语音输出都自动包装为SSML
- lambda事件和上下文对象完全可以通过`this.event`和`this.context`获得。
- 能够覆盖内置函数，使您在管理状态或构建响应方面具有更大的灵活性。例如，将状态属性保存为AWS S3。


## 安装指南
可以立即在Github上使用alexa-sdk，并且可以使用在Node包管理器环境中运行以下命令，进行部署在nodeJs中：
```
npm install --save alexa-sdk
```
## 入门篇：编写Hello World Skill
### 基本项目结构
你的HelloWorld skill需要：
- 入口点，你需要导入skill所需的所有软件包，接收事件，设置appId，设置dynamoDB表，注册处理程序等;
- 管理每个请求的处理函数。

> 入口点 - 程序的入口处，开始处

### 设置入口点
要在您自己的项目中执行此操作，只需创建一个名为index.js的文件并添加以下内容：
```javascript
const Alexa = require('alexa-sdk');

exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.appId = APP_ID // APP_ID是您的skill ID，可在您创建skill的Amazon开发人员控制台中找到。
    alexa.execute();
};
```
这将导入Alexa -sdk并为我们设置一个Alexa对象。

### 实现处理程序函数
接下来，我们需要处理这些事件和意图。Alexa-sdk简化了功能触发的意图。您可以在索引中实现处理函数。js文件刚刚创建，或者您也可以在单独的文件中写入，然后再导入它们。例如，为“HelloWorldIntent”创建一个处理程序，我们可以通过以下两种方式实现:
```javascript
const handlers = {
    'HelloWorldIntent' : function() {
        //直接发出响应
        this.emit(':tell', 'Hello World!');
    }
};
```
Or
```javascript
const handlers = {
    'HelloWorldIntent' : function() {
        //首先使用responseBuilder构建响应，然后发出
        this.response.speak('Hello World!');
        this.emit(':responseReady');
    }
};
```

Alexa-sdk遵循tell/ask响应方法，用于生成outputBuilder中对应于speak/listen的outputSpeech响应对象。
```javascript
this.emit(':tell', 'Hello World!'); 
this.emit(':ask', 'What would you like to do?', 'Please say that again?');
```
同等于：
```javascript
this.response.speak('Hello World!');
this.emit(':responseReady');

this.response.speak('What would you like to do?')
            .listen('Please say that again?');
this.emit(':responseReady');
```
它们之间的区别：ask/listen和：tell/speak是在tell/speak动作之后，会话结束而不用等待用户提供更多输入。 我们将比较使用响应或使用responseBuilder在下一节中创建响应对象的两种方式。

处理程序可以将请求转发给对方，从而可以将处理程序链接在一起以提高用户流量。 下面是一个例子，我们的LaunchRequest和IntentRequest（HelloWorldIntent）都返回相同的'Hello World'消息。

```javascript
const handlers = {
    'LaunchRequest': function () {
    	this.emit('HelloWorldIntent');
	},

	'HelloWorldIntent': function () {
    	this.emit(':tell', 'Hello World!');
	}
};
```


一旦我们设置了事件处理程序，就需要使用我们刚创建的alexa对象的registerHandlers函数来注册它们。 因此，在我们创建的index.js文件中，添加以下内容：

```javascript
const Alexa = require('alexa-sdk');

exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.registerHandlers(handlers);
    alexa.execute();
};
```



















































您也可以一次注册多个处理程序对象：

```javascript
alexa.registerHandlers(handlers, handlers2, handlers3, ...);
```

完成上述步骤后，您的skill应该在设备上正常工作。

## Response vs ResponseBuilder

目前，有两种方法来生成
[响应对象](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#Response%20Format)  在Node.js SDK中。 第一种方法是使用格式遵循的格式 this.emit(`:${action}`, 'responseContent').  
以下是常见 Skill 回答的完整列表示例：

|响应语法 | 描述 |
|----------------|-----------|
| this.emit(':tell',speechOutput);|告诉 [speechOutput](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#outputspeech-object)| 
|this.emit(':ask', speechOutput, repromptSpeech);|询问 [speechOutput](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#outputspeech-object) 和 [repromptSpeech](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#reprompt-object)|
|this.emit(':tellWithCard', speechOutput, cardTitle, cardContent, imageObj);| 告诉 [speechOutput](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#outputspeech-object) 和 [standard card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object)|
|this.emit(':askWithCard', speechOutput, repromptSpeech, cardTitle, cardContent, imageObj);| 询问 [speechOutput](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#outputspeech-object), [repromptSpeech](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#reprompt-object) 和 [standard card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object)|
|this.emit(':tellWithLinkAccountCard', speechOutput);|  告诉 [linkAccount card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object), 有关更多信息，请单击 [这里](https://developer.amazon.com/docs/custom-skills/link-an-alexa-user-with-a-user-in-your-system.html)|
|this.emit(':askWithLinkAccountCard', speechOutput);| 询问 [linkAccount card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object), 有关更多信息，请单击 [这里](https://developer.amazon.com/docs/custom-skills/link-an-alexa-user-with-a-user-in-your-system.html)|
|this.emit(':tellWithPermissionCard', speechOutput, permissionArray);| 告诉 [permission card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#session-object),有关更多信息，请单击 [这里](https://developer.amazon.com/docs/custom-skills/configure-permissions-for-customer-information-in-your-skill.html)|
|this.emit(':askWithPermissionCard', speechOutput, repromptSpeech, permissionArray)| 询问 [permission card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#session-object), 有关更多信息，请单击 [这里](https://developer.amazon.com/docs/custom-skills/configure-permissions-for-customer-information-in-your-skill.html)|
|this.emit(':delegate', updatedIntent);|回应 [delegate directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#delegate)  在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':elicitSlot', slotToElicit, speechOutput, repromptSpeech, updatedIntent);|回应 [elicitSlot directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#elicitslot) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':elicitSlotWithCard', slotToElicit, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj);| 回应 [card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object) 和 [elicitSlot directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#elicitslot) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':confirmSlot', slotToConfirm, speechOutput, repromptSpeech, updatedIntent);|回应 [confirmSlot directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#confirmslot) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':confirmSlotWithCard', slotToConfirm, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj);| 回应 [card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object) 和 [confirmSlot directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#confirmslot) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':confirmIntent', speechOutput, repromptSpeech, updatedIntent);|回应 [confirmIntent directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#confirmintent) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':confirmIntentWithCard', speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj);| 回应 [card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object) 和 [confirmIntent directive](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#confirmintent) 在 [dialog model](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html#dialog-model-required)|
|this.emit(':responseReady');|在响应建立之后但在返回到Alexa服务之前调用。 调用：saveState。 可以被覆盖。|
|this.emit(':saveState', false);|处理将this.attributes的内容和当前处理程序状态保存到DynamoDB，然后将以前生成的响应发送到Alexa服务。 如果您希望使用不同的持久性提供程序，则覆盖。 第二个属性是可选的，可以设置为'true'来强制保存。|
|this.emit(':saveStateError'); |在保存状态时调用是否有错误。 重写以自己处理任何错误。|


如果你想手动创建你自己的回复，你可以使用`this.response`来提供帮助。 `this.response`包含一系列函数，可以用来设置响应的不同属性。 这使您可以利用 Alexa Skill Kit (工具包)内置的音频和视频播放器支持。 一旦你设置了你的回复，你可以调用 `this.emit(':responseReady')` 将你的回复发送给Alexa。 `this.response`中的函数也是可链接的，所以你可以使用尽可能多的行。
以下是使用responseBuilder创建响应的完整列表示例。

| 响应语法 | 描述 |
|----------------|-----------|
|this.response.speak(speechOutput);| 将第一个语音输出设置为 [speechOutput](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#outputspeech-object)|
|this.response.listen(repromptSpeech);| 将重新提示语音输出设置为 [repromptSpeech](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#reprompt-object), shouldEndSession为false。 除非调用此函数，否则this.response会将shouldEndSession设置为true。|
|this.response.cardRenderer(cardTitle, cardContent, cardImage);| 添加一个 [standard card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object) 与cardTitle，cardContent和cardImage相对应|
|this.response.linkAccountCard();| 添加一个 [linkAccount card](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#card-object) 作为回应，有关更多信息，请单击[这里](https://developer.amazon.com/docs/custom-skills/link-an-alexa-user-with-a-user-in-your-system.html)|
|this.response.askForPermissionsConsentCard(permissions);| 添加一张卡要求获得回复的许可 [perimission](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#session-object) 有关更多信息，请单击 [这里](https://developer.amazon.com/docs/custom-skills/configure-permissions-for-customer-information-in-your-skill.html)|
|this.response.audioPlayer(directiveType, behavior, url, token, expectedPreviousToken, offsetInMilliseconds);(Deprecated) | 添加一个 [AudioPlayer directive](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html) 并提供参数作为响应。|
|this.response.audioPlayerPlay(behavior, url, token, expectedPreviousToken, offsetInMilliseconds);| 添加一个 [AudioPlayer directive](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html) 使用提供的参数，并进行设置  [`AudioPlayer.Play`](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html#play) 作为指示类型。|
|this.response.audioPlayerStop();| 添加一个 [AudioPlayer.Stop directive](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html#stop)|
|this.response.audioPlayerClearQueue(clearBehavior);|添加一个 [AudioPlayer.ClearQueue directive](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html#clearqueue) 并设置指令的明确行为。|
|this.response.renderTemplate(template);| 添加一个 [Display.RenderTemplate directive](https://developer.amazon.com/docs/custom-skills/display-interface-reference.html) 作为回应|
|this.response.hint(hintText, hintType);| 添加一个 [Hint directive](https://developer.amazon.com/docs/custom-skills/display-interface-reference.html#hint-directive) 作为回应|
|this.response.playVideo(videoSource, metadata);|添加一个 [VideoApp.Play directive](https://developer.amazon.com/docs/custom-skills/videoapp-interface-reference.html#videoapp-directives) 作为回应|
|this.response.shouldEndSession(bool);| 手动设置shouldEndSession|

当您完成响应设置后，只需调用 `this.emit(':responseReady')` 发送您的响应。
下面是两个用几个响应对象构建响应的示例：

```
//例 1
this.response.speak(speechOutput)
            .listen(repromptSpeech);
this.emit(':responseReady');
//例 2
this.response.speak(speechOutput)
            .cardRenderer(cardTitle, cardContent, cardImage)
            .renderTemplate(template)
            .hint(hintText, hintType);
this.emit(':responseReady');
```
由于responseBuilder构建丰富的响应对象更加灵活，我们更愿意使用此方法来构建响应。

### 提示
- 当发出任何响应事件 `:ask`, `:tell`, `:askWithCard`,等时，如果开发人员没有传递`callback`函数，则调用lambda context.succeed（）方法 立即停止处理任何进一步的后台任务。 任何仍然不完整的异步作业都不会完成，并且响应发出语句下的任何代码行都不会被执行。 对于像`:saveState`这样的非响应事件，情况并非如此。


- 为了将请求从一个状态处理器“转移”到另一个称为意向转发的请求，需要将`this.handler.state`设置为目标状态的名称。 如果目标状态为""，则应调用`this.emit("TargetHandlerName")` 对于任何其他状态，必须调用`this.emitWithState("TargetHandlerName")`.
- 提示值和重新提示值的内容被包装在SSML标签中。 这意味着值中的任何特殊XML字符都需要进行转义编码。 例如，this.emit(":ask", "I like M&M's") 会导致失败，因为`&`字符需要编码为`&amp;`。 其他需要编码的字符包括：`<` 转 `&lt;`, 以及 `>` 转 `&gt;`.


## 标准请求和响应
Alexa通过SSL / TLS上的HTTP通过请求 - 响应机制与 Skill 服务进行通信。 当用户与 Alexa Skill 交互时，您的服务会收到包含JSON正文的POST请求。 请求主体包含服务执行其逻辑并生成JSON格式响应所需的参数。 由于Node.js可以本地处理JSON，所以Alexa Node.js SDK不需要执行JSON序列化和反序列化。 开发人员只负责提供适当的响应对象，以便Alexa响应客户请求。 请求主体的JSON结构的文档可以在[这里](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#request-format)找到.

SpeechletResponse可能包含以下属性：
- OutputSpeech
- Reprompt
- Card
- List of Directives
- shouldEndSession

作为一个例子，一个包含言语和卡片的简单回应可以被构造如下：

```javascript
const speechOutput = 'Hello world!';
const repromptSpeech = 'Hello again!';
const cardTitle = 'Hello World Card';
const cardContent = 'This text will be displayed in the companion app card.';
const imageObj = {
	smallImageUrl: 'https://imgs.xkcd.com/comics/standards.png',
	largeImageUrl: 'https://imgs.xkcd.com/comics/standards.png'
};
this.response.speak(speechOutput)
            .listen(repromptSpeech)
            .cardRenderer(cardTitle, cardContent, imageObj);
this.emit(':responseReady');
```


## 接口 

### AudioPlayer 接口
开发人员可以在他们的 Skill 响应中包含以下指令（分别）
- `PlayDirective`
- `StopDirective`
- `ClearQueueDirective`

这是一个使用`PlayDirective`来流式传输音频的例子：
```javascript
const handlers = {
    'LaunchRequest' : function() {
        const speechOutput = 'Hello world!';
        const behavior = 'REPLACE_ALL';
        const url = 'https://url/to/audiosource';
        const token = 'myMusic';
        const expectedPreviousToken = 'expectedPreviousStream';
        const offsetInMilliseconds = 10000;
        this.response.speak(speechOutput)
                    .audioPlayerPlay(behavior, url, token, expectedPreviousToken, offsetInMilliseconds);
        this.emit(':responseReady');
    }
};
```
在上面的例子中，Alexa会先说`speechOutput`，然后尝试播放音频。

在构建利用[AudioPlayer](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html) 接口的 Skill 时，会发送“回放”请求以通知技术人员关于“回放”状态的更改。您可以为其各自的事件实施处理函数。
```javascript
const handlers = {
    'PlaybackStarted' : function() {
    	console.log('Alexa begins playing the audio stream');
    },
    'PlaybackFinished' : function() {
    	console.log('The stream comes to an end');
    },
    'PlaybackStopped' : function() {
    	console.log('Alexa stops playing the audio stream');
    },
    'PlaybackNearlyFinished' : function() {
    	console.log('The currently playing stream is nearly complate and the device is ready to receive a new stream');
    },
    'PlaybackFailed' : function() {
    	console.log('Alexa encounters an error when attempting to play a stream');
    }
};
```


有关`AudioPlayer`界面的其他文档可以在[这里](https://developer.amazon.com/docs/custom-skills/audioplayer-interface-reference.html)找到.

注意：关于`imgObj`的规格请看[这里](https://developer.amazon.com/docs/custom-skills/include-a-card-in-your-skills-response.html#creating-a-home-card-to-display-text-and-an-image)

### 对话界面

'Dialog`界面提供了用于管理 Skill 和用户之间的多回合对话的指令。 您可以使用这些指令向用户询问您需要的信息以完成他们的请求。 请参阅[对话框界面](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/dialog-interface-reference) 和 [Skill Editor](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/ask-define-the-vui-with-gui) 有关如何使用对话框指令的更多信息。

你可以使用`this.event.request.dialogState`来访问当前的`dialogState`。

#### 代表指令
发送Alexa命令以处理与用户对话的下一个回合。 如果 Skill 具有对话模型并且对话框（`dialogState`）的当前状态是 `STARTED` 或 `IN_PROGRESS`，则可以使用此指令。 如果`dialogState`为`COMPLETED`，则不能发出此指令。
你可以使用 `this.emit(':delegate')` 发送委托指令响应。
```javascript
const handlers = {
    'BookFlightIntent': function () {
        if (this.event.request.dialogState === 'STARTED') {
            let updatedIntent = this.event.request.intent;
            // Pre-fill slots: update the intent object with slot values for which
            // 预填充Slot：使用Slot值更新意图对象
            // you have defaults, then emit :delegate with this updated intent.
            // 你有默认值，然后发出：委托与此更新的意图。
            updatedIntent.slots.SlotName.value = 'DefaultValue';
            this.emit(':delegate', updatedIntent);
        } else if (this.event.request.dialogState !== 'COMPLETED'){
            this.emit(':delegate');
        } else {
            // All the slots are filled (And confirmed if you choose to confirm slot/intent)
            // 所有Slot都已填充（并且如果您选择确认Slot/intent，请确认）
            handlePlanMyTripIntent();
        }
    }
};
```

#### 引出Slot指令
向Alexa发送一个命令，询问用户特定Slot的值。 在`slotToElicit`中指定要引出的Slot的名称。 提供一个提示，要求用户输入`speechOutput`中的Slot值。

您可以使用 `this.emit(':elicitSlot', slotToElicit, speechOutput, repromptSpeech, updatedIntent)` 或 `this.emit(':elicitSlotWithCard', slotToElicit, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)`发送Slot指示响应。

使用 `this.emit(':elicitSlotWithCard', slotToElicit, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)`时, `updatedIntent` 和 `imageObj` 是可选参数。 您可以将它们设置为“null”或不传递它们。
```javascript
const handlers = {
    'BookFlightIntent': function () {
        const intentObj = this.event.request.intent;
        if (!intentObj.slots.Source.value) {
            const slotToElicit = 'Source';
            const speechOutput = 'Where would you like to fly from?';
            const repromptSpeech = speechOutput;
            this.emit(':elicitSlot', slotToElicit, speechOutput, repromptSpeech);
        } else if (!intentObj.slots.Destination.value) {
            const slotToElicit = 'Destination';
            const speechOutput = 'Where would you like to fly to?';
            const repromptSpeech = speechOutput;
            const cardContent = 'What is the destination?';
            const cardTitle = 'Destination';
            const updatedIntent = intentObj;
            // An intent object representing the intent sent to your skill.
            // 表示发送给您 Skill 的意图的意图对象。
            // You can use this property set or change slot values and confirmation status if necessary.
            // 如有必要，您可以使用此属性集或更改Slot值和确认状态。
            const imageObj = {
                smallImageUrl: 'https://imgs.xkcd.com/comics/standards.png',
                largeImageUrl: 'https://imgs.xkcd.com/comics/standards.png'
            };
            this.emit(':elicitSlotWithCard', slotToElicit, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj);
        } else {
            handlePlanMyTripIntentAllSlotsAreFilled();
        }
    }
};
```

#### 确认Slot指令
在继续进行对话之前，发送Alexa命令以确认特定slot的值。 在`slotToConfirm`中指定要确认的slot的名称。 提供一个提示，要求用户在`speechOutput`中进行确认。

您可以使用 `this.emit(':confirmSlot', slotToConfirm, speechOutput, repromptSpeech, updatedIntent)` 或 `this.emit(':confirmSlotWithCard', slotToConfirm, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)` 发送确认slot指令响应。

使用 `this.emit(':confirmSlotWithCard', slotToConfirm, speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)`时, `updatedIntent` 和 `imageObj` 是可选参数。 您可以将它们设置为“null”或不传递它们。
```javascript
const handlers = {
    'BookFlightIntent': function () {
        const intentObj = this.event.request.intent;
        if (intentObj.slots.Source.confirmationStatus !== 'CONFIRMED') {
            if (intentObj.slots.Source.confirmationStatus !== 'DENIED') {
                // Slot value is not confirmed
                // Slot值未被确认
                const slotToConfirm = 'Source';
                const speechOutput = 'You want to fly from ' + intentObj.slots.Source.value + ', is that correct?';
                const repromptSpeech = speechOutput;
                this.emit(':confirmSlot', slotToConfirm, speechOutput, repromptSpeech);
            } else {
                // Users denies the confirmation of slot value
                // 用户拒绝确认slot值
                const slotToElicit = 'Source';
                const speechOutput = 'Okay, Where would you like to fly from?';
                this.emit(':elicitSlot', slotToElicit, speechOutput, speechOutput);
            }
        } else if (intentObj.slots.Destination.confirmationStatus !== 'CONFIRMED') {
            if (intentObj.slots.Destination.confirmationStatus !== 'DENIED') {
                const slotToConfirm = 'Destination';
                const speechOutput = 'You would like to fly to ' + intentObj.slots.Destination.value + ', is that correct?';
                const repromptSpeech = speechOutput;
                const cardContent = speechOutput;
                const cardTitle = 'Confirm Destination';
                this.emit(':confirmSlotWithCard', slotToConfirm, speechOutput, repromptSpeech, cardTitle, cardContent);
            } else {
                const slotToElicit = 'Destination';
                const speechOutput = 'Okay, Where would you like to fly to?';
                const repromptSpeech = speechOutput;
                this.emit(':elicitSlot', slotToElicit, speechOutput, repromptSpeech);
            }
        } else {
            handlePlanMyTripIntentAllSlotsAreConfirmed();
        }
    }
};
```

#### 确认意向指令
发送Alexa命令以确认在 Skill 采取行动之前用户为意图提供的所有信息。 提供一个提示，要求用户在`speechOutput`中进行确认。 确保在提示中重复用户需要确认的所有值。

您可以使用 `this.emit(':confirmIntent', speechOutput, repromptSpeech, updatedIntent)` 或 `this.emit(':confirmIntentWithCard', speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)`发送确认意图指示响应。

使用 `this.emit(':confirmIntentWithCard', speechOutput, repromptSpeech, cardTitle, cardContent, updatedIntent, imageObj)`时, `updatedIntent` 和 `imageObj` 是可选参数。 您可以将它们设置为“null”或不传递它们。
```javascript
const handlers = {
    'BookFlightIntent': function () {
        const intentObj = this.event.request.intent;
        if (intentObj.confirmationStatus !== 'CONFIRMED') {
            if (intentObj.confirmationStatus !== 'DENIED') {
                // Intent is not confirmed
                // 意图未被确认
                const speechOutput = 'You would like to book flight from ' + intentObj.slots.Source.value + ' to ' +
                intentObj.slots.Destination.value + ', is that correct?';
                const cardTitle = 'Booking Summary';
                const repromptSpeech = speechOutput;
                const cardContent = speechOutput;
                this.emit(':confirmIntentWithCard', speechOutput, repromptSpeech, cardTitle, cardContent);
            } else {
                // Users denies the confirmation of intent. May be value of the slots are not correct.
                // 用户拒绝确认意图。 可能是slots的值不正确。
                handleIntentConfimationDenial();
            }
        } else {
            handlePlanMyTripIntentAllSlotsAndIntentAreConfirmed();
        }
    }
};
```

有关`Dialog`界面的其他文档可以在[这里](https://developer.amazon.com/docs/custom-skills/dialog-interface-reference.html)找到.

### 显示界面
Alexa提供了多种`Display templates`来支持各种演示。 目前，有两类`Display templates`：
> Display templates  - 显示模板
- `BodyTemplate` 显示无法选择的文字和图像。 目前有四个选项
+ `BodyTemplate1`
+ `BodyTemplate2`
+ `BodyTemplate3`
+ `BodyTemplate6`
+ `BodyTemplate7`
- `ListTemplate` 显示可滚动的项目列表，每个项目都有相关的文本和可选图像。 这些图像可以选择。 目前有两种选择：
+ `ListTemplate1`
+ `ListTemplate2`


开发人员必须在其skill响应中包含 `Display.RenderTemplate` 指令。
模板构建器现在包含在 templateBuilders 命名空间的 alexa-sdk 中。 这些提供了一组帮助器方法来为`Display.RenderTemplate`指令构建JSON模板。 在下面的例子中，我们使用`BodyTemplate1Builder`来构建`Body template`。

```javascript
const Alexa = require('alexa-sdk');
// utility methods for creating Image and TextField objects
// 用于创建Image和TextField对象的实用方法
const makePlainText = Alexa.utils.TextUtils.makePlainText;
const makeImage = Alexa.utils.ImageUtils.makeImage;

// ...
'LaunchRequest' : function() {
	const builder = new Alexa.templateBuilders.BodyTemplate1Builder();

	const template = builder.setTitle('My BodyTemplate1')
							.setBackgroundImage(makeImage('http://url/to/my/img.png'))
							.setTextContent(makePlainText('Text content'))
							.build();

	this.response.speak('Rendering a body template!')
				.renderTemplate(template);
	this.emit(':responseReady');
}
```

We've added helper utility methods to build Image and TextField objects. They are located in the `Alexa.utils` namespace.

```javascript
const ImageUtils = require('alexa-sdk').utils.ImageUtils;

// Outputs an image with a single source
// 用单一来源输出图像
ImageUtils.makeImage(url, widthPixels, heightPixels, size, description);
/**
Outputs {
    contentDescription : '<description>'
    sources : [
        {
            url : '<url>',
            widthPixels : '<widthPixels>',
            heightPixels : '<heightPixels>',
            size : '<size>'
        }
    ]
}
*/

ImageUtils.makeImages(imgArr, description);
/**
Outputs {
    contentDescription : '<description>'
    sources : <imgArr> // array of {url, size, widthPixels, heightPixels}
}
*/


const TextUtils = require('alexa-sdk').utils.TextUtils;

TextUtils.makePlainText('my plain text field');
/**
Outputs {
    text : 'my plain text field',
    type : 'PlainText'
}
*/

TextUtils.makeRichText('my rich text field');
/**
Outputs {
    text : 'my rich text field',
    type : 'RichText'
}
*/

```
在下一个例子中，我们将使用ListTemplate1Builder和ListItemBuilder来构建ListTemplate1。
```javascript
const Alexa = require('alexa-sdk');
const makePlainText = Alexa.utils.TextUtils.makePlainText;
const makeImage = Alexa.utils.ImageUtils.makeImage;
// ...
'LaunchRequest' : function() {
    const itemImage = makeImage('https://url/to/imageResource', imageWidth, imageHeight);
    const listItemBuilder = new Alexa.templateBuilders.ListItemBuilder();
    const listTemplateBuilder = new Alexa.templateBuilders.ListTemplate1Builder();
    listItemBuilder.addItem(itemImage, 'listItemToken1', makePlainText('List Item 1'));
    listItemBuilder.addItem(itemImage, 'listItemToken2', makePlainText('List Item 2'));
    listItemBuilder.addItem(itemImage, 'listItemToken3', makePlainText('List Item 3'));
    listItemBuilder.addItem(itemImage, 'listItemToken4', makePlainText('List Item 4'));
    const listItems = listItemBuilder.build();
    const listTemplate = listTemplateBuilder.setToken('listToken')
    										.setTitle('listTemplate1')
    										.setListItems(listItems)
    										.build();
    this.response.speak('Rendering a list template!')
    			.renderTemplate(listTemplate);
    this.emit(':responseReady');
}
```

发送一个`Display.RenderTemplate`指令给一个无头设备（比如echo）会导致一个无效的指令错误被抛出。 要检查设备是否支持特定指令，可以检查设备的 supportedInterfaces 属性。

```javascript
const handler = {
    'LaunchRequest' : function() {

    this.response.speak('Hello there');

    // Display.RenderTemplate directives can be added to the response
    // Display.RenderTemplate 指令可以添加到响应中
    if (this.event.context.System.device.supportedInterfaces.Display) {
        //... build mytemplate using TemplateBuilder
        //... 使用TemplateBuilder构建模板
        this.response.renderTemplate(myTemplate);
    }

    this.emit(':responseReady');
    }
};
```

Similarly for video, you check if VideoApp is a supported interface of the device

```javascript
const handler = {
    'PlayVideoIntent' : function() {

    // VideoApp.Play directives can be added to the response
    // VideoApp.Play 指令可以添加到响应中
    if (this.event.context.System.device.supportedInterfaces.VideoApp) {
        this.response.playVideo('http://path/to/my/video.mp4');
    } else {
        this.response.speak("The video cannot be played on your device. " +
        "To watch this video, try launching the skill from your echo show device.");
    }

        this.emit(':responseReady');
    }
};
```
Additional documentation on `Display` interface can be found [here](https://developer.amazon.com/docs/custom-skills/display-interface-reference.html).

### Playback Controller Interface
The `PlaybackController` interface enables skills to handles requests sent when a customer interacts with player controls such as buttons on a device or a remote control. Those requests are different from normal voice requests such as "Alexa, next song" which are standard intent requests. In order to enable skill to handle `PlaybackController` requests, developers must implement `PlaybackController` interface in Alexa Node.js SDK.
```javascript
const handlers = {
    'NextCommandIssued' : function() {
        //Your skill can respond to NextCommandIssued with any AudioPlayer directive.
        //您的skill可以通过任何AudioPlayer指令响应NextCommandIssued。
    },
    'PauseCommandIssued' : function() {
        //Your skill can respond to PauseCommandIssued with any AudioPlayer directive.
        //您的skill可以通过任何AudioPlayer指令响应PauseCommandIssued。
    },
    'PlayCommandIssued' : function() {
        //Your skill can respond to PlayCommandIssued with any AudioPlayer directive.
        //您的skill可以通过任何AudioPlayer指令响应PlayCommandIssued。
    },
    'PreviousCommandIssued' : function() {
        //Your skill can respond to PreviousCommandIssued with any AudioPlayer directive.
        //您的skill可以通过任何AudioPlayer指令响应PreviousCommand。
    },
    'System.ExceptionEncountered' : function() {
        //Your skill cannot return a response to System.ExceptionEncountered.
        //您的skill无法返回对System.ExceptionEncountered的响应。
    }
};
```
有关`PlaybackController`界面的其他文档可以在[这里](https://developer.amazon.com/docs/custom-skills/playback-controller-interface-reference.html)找到.


### VideoApp界面

要在Echo Show上流式传输本地视频文件，开发人员必须发送`VideoApp.Launch`指令。 Alexa Node.js SDK在responseBuilder中提供了一个函数来帮助构建JSON响应对象。
以下是流式传输视频的示例：
```javascript
//...
'LaunchRequest' : function() {
    const videoSource = 'https://url/to/videosource';
    const metadata = {
    	'title': 'Title for Sample Video',
    	'subtitle': 'Secondary Title for Sample Video'
    };
    this.response.playVideo(videoSource, metadata);
    this.emit(':responseReady');
}
```
有关`VideoApp`界面的其他文档可以在[这里](https://developer.amazon.com/docs/custom-skills/videoapp-interface-reference.html)找到.

### Skill and List 事件
 Skill 开发人员有能力直接与 Alexa Skill 活动进行整合。 如果 Skill 订阅了这些事件，则在事件发生时通知 Skill 。
为了在您的 Skill 服务中使用活动，您必须按照[通过SMAPI向您的 Skill 添加活动](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/add-events-to-your-skill-with-smapi)中所述设置对Alexa skill管理API（SMAPI）的访问权限，.


 Skill 和列表事件会议结束。 一旦你的 Skill 被设置为接收这些事件。 您可以通过将事件名称添加到默认事件处理程序来指定行为。

```javascript
const handlers = {
    'AlexaSkillEvent.SkillEnabled' : function() {
        const userId = this.event.context.System.user.userId;
        console.log(`skill was enabled for user: ${userId}`);
    },
    'AlexaHouseholdListEvent.ItemsCreated' : function() {
        const listId = this.event.request.body.listId;
        const listItemIds = this.event.request.body.listItemIds;
        console.log(`The items: ${JSON.stringify(listItemIds)} were added to list ${listId}`);
    },
    'AlexaHouseholdListEvent.ListCreated' : function() {
        const listId = this.event.request.body.listId;
        console.log(`The new list: ${JSON.stringify(listId)} was created`);
    }
    //...
};

exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.registerHandlers(handlers);
    alexa.execute();
};
```
我们已经创建了一个[示例技巧和步骤](https://github.com/Alexa/alexa-cookbook/tree/master/context/skill-events) 来指导您完成skill事件的订阅过程

## 服务

### Device Address Service - 设备地址服务

Alexa NodeJS SDK提供了一个```DeviceAddressService```辅助类，它利用设备地址API来检索客户设备地址信息。 目前提供了以下方法：

```javascript
getFullAddress(deviceId, apiEndpoint, token)
getCountryAndPostalCode(deviceId, apiEndpoint, token)
```
``apiEndpoint`` and ``token`` can be retrieved from the request at ``this.event.context.System.apiEndpoint`` and ``this.event.context.System.user.permissions.consentToken``

``deviceId`` can also be retrieved from request at ``this.event.context.System.device.deviceId``

```javascript
const Alexa = require('alexa-sdk');

'DeviceAddressIntent': function () {
    if (this.event.context.System.user.permissions) {
        const token = this.event.context.System.user.permissions.consentToken;
        const apiEndpoint = this.event.context.System.apiEndpoint;
        const deviceId = this.event.context.System.device.deviceId;

        const das = new Alexa.services.DeviceAddressService();
        das.getFullAddress(deviceId, apiEndpoint, token)
        .then((data) => {
            this.response.speak('<address information>');
            console.log('Address get: ' + JSON.stringify(data));
            this.emit(':responseReady');
        })
        .catch((error) => {
            this.response.speak('I\'m sorry. Something went wrong.');
            this.emit(':responseReady');
            console.log(error.message);
        });
    } else {
        this.response.speak('Please grant skill permissions to access your device address.');
        this.emit(':responseReady');
    }
}
```


### List Management Service - 列表管理服务


Alexa客户可以访问两个默认列表：Alexa待办事项和Alexa购物。 另外，
Alexa客户可以创建和管理,自定义列表中支持该skill的skill。

Alexa NodeJS SDK提供了一个```ListManagementService```辅助类来帮助开发人员创建更容易管理默认和自定义Alexa列表的 Skill 。 目前提供了以下方法：


````javascript
getListsMetadata(token)
createList(listObject, token)
getList(listId, itemStatus, token)
updateList(listId, listObject, token)
deleteList(listId, token)
createListItem(listId, listItemObject, token)
getListItem(listId, itemId, token)
updateListItem(listId, itemId, listItemObject, token)
deleteListItem(listId, itemId, token)
````

``token`` can be retrieved from the request at ``this.event.context.System.user.permissions.consentToken``

``listId`` can be retrieved from a ``GetListsMetadata`` call.
``itemId`` can be retrieved from a ``GetList`` call

````javascript
const Alexa = require('alexa-sdk');

function getListsMetadata(token) {
    const lms = new Alexa.services.ListManagementService();
    lms.getListsMetadata(token)
    .then((data) => {
        console.log('List retrieved: ' + JSON.stringify(data));
        this.context.succeed();
    })
    .catch((error) => {
        console.log(error.message);
    });
};
````

### Directive Service

`enqueue(directive, endpoint, token)`

在skill执行期间异步地向Alexa设备返回指令。 目前它只接受讲话指令，同时支持SSML（包括MP3音频）和纯文本输出格式。 指令只能在skill激活时返回给始发设备。 可以分别从`this.event.context.System.apiEndpoint`和`this.event.context.System.apiAccessToken`的请求中检索`apiEndpoint`和`token`参数。
- 回复发言应限制在600个字符以内。
- SSML中引用的任何音频片段应限制为30秒。
- skill可以通过指令服务发送的指令数量没有限制。 如有必要，skill可以为每次执行发送多个请求。
- 指令服务不包含任何重复数据删除处理，因此我们不建议进行任何形式的重试处理，因为它可能会导致用户多次收到相同的指令。

```javascript
const Alexa = require('alexa-sdk');

const handlers = {
    'SearchIntent' : function() {
        const requestId = this.event.request.requestId;
        const token = this.event.context.System.apiAccessToken;
        const endpoint = this.event.context.System.apiEndpoint;
        const ds = new Alexa.services.DirectiveService();

        const directive = new Alexa.directives.VoicePlayerSpeakDirective(requestId, "Please wait...");
        const progressiveResponse = ds.enqueue(directive, endpoint, token)
        .catch((err) => {
            // catch API errors so skill processing an continue
            // 捕获API错误，以便继续处理 Skill 
        });
        const serviceCall = callMyService();

        Promise.all([progressiveResponse, serviceCall])
        .then(() => {
            this.response.speak('I found the following results');
            this.emit(':responseReady');
        });
    }
};

```

## Extend Features

### Skill State Management



Alexa-sdk使用状态管理器将传入的意图路由到正确的函数处理程序。 状态作为字符串存储在会话属性中，指示skill的当前状态。 在定义意图处理程序时，您可以通过将状态字符串附加到意图名称来模拟内置意向路由，但alexa-sdk可以帮助您完成。

我们以一个示例 skill [highlowgame](https://github.com/alexa/skill-sample-nodejs-highlowgame/blob/master/lambda/custom/index.js) 为例来说明状态管理在SDK中的工作原理。 在这项skill中，客户会猜测一个数字，Alexa会告诉他们数字是高还是低。 它也会告诉客户玩过多少次。 它有两个状态`start`'开始' 和 `guess`'猜测'

```javascript
const states = {
    GUESSMODE: '_GUESSMODE', // User is trying to guess the number.
                             // 用户正在猜测该号码。
    STARTMODE: '_STARTMODE' // Prompt the user to start or restart the game.
                            // 提示用户启动或重新启动游戏。
};
```

newSessionHandlers中的NewSession处理程序将快速切入任何传入的意图或启动请求，并将它们路由到此处理程序。
```javascript
const newSessionHandlers = {
    'NewSession': function() {
        if(Object.keys(this.attributes).length === 0) { // Check if it's the first time the skill has been invoked
        // 检查是否是第一次调用skill
            this.attributes['endedSessionCount'] = 0;
            this.attributes['gamesPlayed'] = 0;
        }
        this.handler.state = states.STARTMODE;
        this.response.speak('Welcome to High Low guessing game. You have played '
                        + this.attributes['gamesPlayed'].toString() + ' times. Would you like to play?')
                    .listen('Say yes to start the game or no to quit.');
        this.emit(':responseReady');
    }
};
```
请注意，当创建一个新会话时，我们只需使用`this.handler.state`将我们的skill状态设置为`STARTMODE`。 如果您设置了DynamoDB表，skill状态将自动保留在skill的会话属性中，并且可以选择在会话中持久保存。

还有一点很重要，那就是`NewSession`是一个很好的通用行为和一个很好的切入点，但它并不是必需的。 只有定义了具有该名称的处理程序时，才会调用`NewSession`。 您定义的每个状态都可以拥有自己的`NewSession`处理程序，如果您使用的是内置持久性，则会调用它。 在上面的例子中，我们可以为`states.STARTMODE`和`states.GUESSMODE`定义不同的`NewSession`行为，这给我们增加了灵活性。


为了定义将响应我们skill的不同状态的意图，我们需要使用`Alexa.CreateStateHandler`函数。 此处定义的任何意图处理程序只有在该skill处于特定状态时才会起作用，这使我们具有更大的灵活性！

例如，如果我们处于上面定义的`GUESSMODE`状态，我们想要处理用户对问题的回应。 这可以使用像这样的StateHandlers来完成：

```javascript
const guessModeHandlers = Alexa.CreateStateHandler(states.GUESSMODE, {

'NewSession': function () {
    this.handler.state = '';
    this.emitWithState('NewSession'); // Equivalent to the Start Mode NewSession handler
    // 等同于启动模式NewSession处理程序
},

'NumberGuessIntent': function() {
    const guessNum = parseInt(this.event.request.intent.slots.number.value);
    const targetNum = this.attributes['guessNumber'];

    console.log('user guessed: ' + guessNum);

    if(guessNum > targetNum){
        this.emit('TooHigh', guessNum);
    } else if( guessNum < targetNum){
        this.emit('TooLow', guessNum);
    } else if (guessNum === targetNum){
        // With a callback, use the arrow function to preserve the correct 'this' context
        // 通过回调，使用箭头功能来保留正确的 'this' 上下文
        this.emit('JustRight', () => {
            this.response.speak(guessNum.toString() + 'is correct! Would you like to play a new game?')
                        .listen('Say yes to start a new game, or no to end the game.');
            this.emit(':responseReady');
        });
    } else {
        this.emit('NotANum');
    }
},

'AMAZON.HelpIntent': function() {
    this.response.speak('I am thinking of a number between zero and one hundred, try to guess and I will tell you' +
    ' if it is higher or lower.')
                .listen('Try saying a number.');
    this.emit(':responseReady');
},

'SessionEndedRequest': function () {
    console.log('session ended!');
    this.attributes['endedSessionCount'] += 1;
    this.emit(':saveState', true); // Be sure to call :saveState to persist your session attributes in DynamoDB
    // 请务必调用：saveState以在DynamoDB中保留会话属性
},

'Unhandled': function() {
    this.response.speak('Sorry, I didn\'t get that. Try saying a number.')
                .listen('Try saying a number.');
    this.emit(':responseReady');
}
});
```
On the flip side, if I am in `STARTMODE` I can define my `StateHandlers` to be the following:

```javascript
const startGameHandlers = Alexa.CreateStateHandler(states.STARTMODE, {

    'NewSession': function () {
        this.emit('NewSession'); // Uses the handler in newSessionHandlers
        // 在newSessionHandlers中使用处理程序
    },

    'AMAZON.HelpIntent': function() {
        const message = 'I will think of a number between zero and one hundred, try to guess and I will tell you if it' +
        ' is higher or lower. Do you want to start the game?';
        this.response.speak(message)
                    .listen(message);
        this.emit(':responseReady');
    },

    'AMAZON.YesIntent': function() {
        this.attributes['guessNumber'] = Math.floor(Math.random() * 100);
        this.handler.state = states.GUESSMODE;
        this.response.speak('Great! ' + 'Try saying a number to start the game.')
                    .listen('Try saying a number.');
        this.emit(':responseReady');
    },

    'AMAZON.NoIntent': function() {
        this.response.speak('Ok, see you next time!');
        this.emit(':responseReady');
    },

    'SessionEndedRequest': function () {
        console.log('session ended!');
        this.attributes['endedSessionCount'] += 1;
        this.emit(':saveState', true);
    },

    'Unhandled': function() {
        const message = 'Say yes to continue, or no to end the game.';
        this.response.speak(message)
                    .listen(message);
        this.emit(':responseReady');
    }
});
```


看一看`guessModeHandlers`对象中没有定义`AMAZON.YesIntent`和`AMAZON.NoIntent`，因为在这种状态下，'yes'或'no'响应没有意义。这些意图将被`Unhandled`处理程序捕获。

另外，请注意在两种状态下`NewSession`和`Unhandled`的不同行为？在这个游戏中，我们通过调用`newSessionHandlers`对象中定义的`NewSession`处理程序来“重置”状态。您也可以跳过定义它，alexa-sdk将调用当前状态的意图处理程序。只要记得在你调用`alexa.execute()`之前注册你的状态处理程序，否则它们将不会被找到。

你的属性会在你结束会话时自动保存，但是如果用户结束会话，你必须发出`:saveState`事件(`this.emit(':saveState', true)`)来强制保存。您应该在您的`SessionEndedRequest`处理程序中执行此操作，该处理程序在用户通过说'退出'或超时结束会话时调用。看看上面的例子。

如果你想明确重置状态，下面的代码应该工作：
```javascript
this.handler.state = '' // delete this.handler.state might cause reference errors
// 删除this.handler.state可能会导致引用错误
delete this.attributes['STATE'];
```

### Persisting Skill Attributes through DynamoDB
### 通过DynamoDB持久保持Skill属性

你们中的许多人想要将会话属性值保存到存储器中以供进一步使用。 Alexa-sdk直接与[Amazon DynamoDB]（https://aws.amazon.com/dynamodb/）（NoSQL数据库服务）集成，使您能够使用一行代码执行此操作。

在那之前，只需在您的alexa对象上设置DynamoDB表的名称即可
Simply set the name of the DynamoDB table on your alexa object before you call alexa.execute.
```javascript
exports.handler = function (event, context, callback) {
    const alexa = Alexa.handler(event, context, callback);
    alexa.appId = appId;
    alexa.dynamoDBTableName = 'YourTableName'; // That's it!
    alexa.registerHandlers(State1Handlers, State2Handlers);
    alexa.execute();
};
```
然后稍后设置一个值，您只需调用 alexa 对象的 attributes 属性即可。 没有更多的单独的`put`和`get`功能！
```javascript
this.attributes['yourAttribute'] = 'value';
```

您可以预先[手动创建表](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.CreateTables.html) ，或者直接给您的 Lambda 函数 DynamoDB [创建表权限](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_CreateTable.html) 并自动发生。 请记住，在第一次调用时创建表可能需要一分钟左右的时间。 如果您手动创建表，则主键必须是名为"userId"的字符串值。

注意：如果您在lambda上托管 Skill 并选择通过DynamoDB保留 Skill 属性，请确保lambda函数的excution角色包含对DynamoDB的访问权限。


### Adding Multi-Language Support for Skill
### 为 Skill 添加多语言支持
我们来看看Hello World示例。 按以下格式定义所有面向用户的语言字符串。
```javascript
const languageStrings = {
    'en-GB': {
        'translation': {
            'SAY_HELLO_MESSAGE' : 'Hello World!'
        }
    },
    'en-US': {
        'translation': {
            'SAY_HELLO_MESSAGE' : 'Hello World!'
        }
    },
    'de-DE': {
        'translation': {
            'SAY_HELLO_MESSAGE' : 'Hallo Welt!'
        }
    }
};
```

要在Alexa-sdk中启用字符串国际化功能，请将资源设置为我们上面创建的对象。
```javascript
exports.handler = function(event, context, callback) {
    const alexa = Alexa.handler(event, context);
    alexa.appId = appId;
    // To enable string internationalization (i18n) features, set a resources object.
    // 要启用字符串国际化（i18n）功能，请设置资源对象。
    alexa.resources = languageStrings;
    alexa.registerHandlers(handlers);
    alexa.execute();
};
```

Once you are done defining and enabling language strings, you can access these strings using the this.t() function. Strings will be rendered in the language that matches the locale of the incoming request.
```javascript
const handlers = {
    'LaunchRequest': function () {
        this.emit('SayHello');
    },
    'HelloWorldIntent': function () {
        this.emit('SayHello');
    },
    'SayHello': function () {
        this.response.speak(this.t('SAY_HELLO_MESSAGE'));
        this.emit(':responseReady');
    }
};
```
有关以多种语言开发和部署 Skill 的更多信息，请参阅[这里](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/developing-skills-in-multiple-languages).

### Device ID Support
### 设备ID支持
当客户启用您的 Alexa skill 时，您的 skill 可以获得客户的许可以使用与客户的Alexa设备关联的地址数据。 然后，您可以使用此地址数据为 skill 提供关键功能，或者增强客户体验。

`deviceId`现在通过每个请求中的上下文对象公开，并且可以通过`this.event.context.System.device.deviceId`在任何意图处理程序中访问。 查看[地址API示例skill](https://github.com/alexa/skill-sample-node-device-address-api) ，了解我们如何利用deviceId和地址API在skill中使用用户的设备地址。

### Speechcons (Interjections)
### Speechcons（感叹词）

[Speechcons](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/speechcon-reference) 是Alexa特别表达的特殊词语和短语。 为了使用它们，你可以在文本中包含SSML标记以发出。

* `this.emit(':tell', 'Sometimes when I look at the Alexa skills you have all taught me, I just have to say, <say-as interpret-as="interjection">Bazinga.</say-as> ');`
* `this.emit(':tell', '<say-as interpret-as="interjection">Oh boy</say-as><break time="1s"/> this is just an example.');`

_Speechcons are supported for English (US), English (UK), English (India), and German._

## Setting up your development environment
## 建立你的开发环境
> 要求 安装 Gulp (自动化构建工具) ， mocha ，npm
- Requirements
- Gulp & mocha  ```npm install -g gulp mocha```
- Run npm install to pull down stuff
- run gulp to run tests/linter

For more information about getting started with the Alexa Skills Kit, check out the following additional assets:
有关开始使用 Alexa Skills Kit 的更多信息，请查看以下附加资源：

[Alexa开发聊天播客](http://bit.ly/alexadevchat)

[Alexa Training with Big Nerd Ranch](https://developer.amazon.com/public/community/blog/tag/Big+Nerd+Ranch)

[Alexa Skills Kit (ASK)](https://developer.amazon.com/ask)

[Alexa开发者论坛](https://forums.developer.amazon.com/forums/category.jspa?categoryID=48)

[培训 Alexa Skills Kit](https://developer.amazon.com/alexa-skills-kit/alexa-skills-developer-training)

-Dave ( [@TheDaveDev](http://twitter.com/thedavedev))
