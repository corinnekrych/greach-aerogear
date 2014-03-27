Hybrid Mobile App in Minutes, Not Days
by Corinne Krych, Fabrice Matrat and Sebastien Blanc

This is not meant to be an exact transcript of the talk but rather a sort of guide.

#### Slide: Hybrid Mobile App in Minutes, Not Days
- Today we will show you how to build Hybrid Mobile App 
- it’ sub-titles fast and furious II, because it’s the second year we (the 3 musketeers) present a mobile talk at Greach.
BUT let's start introducing ourselves
#### Slide: Corinne Presentation
With over 15 years of experience in IT.... 
- I am Athos for the 3musketeers dev Grails Mobile Solution within OS team
- I am cofounder of RGUG
- co-organizerr of JSSophia
- I work AG mobile project, I’m iOS tech lead
- I’m also part of the Duchess team
#### Slide:Sebi Presentation
Seb who is my co-worker at RH, working on AG project, 
co founder RGUG
Aramis for the 3musketeers
charming guys, html5 addicted
#### Slide:Fabrice Presentation
- also co-founder of RivieraGUG - you may have notice by now we’re a bunch of friends
- an active member of JSSophia
- porthos for the 3musketeers 
#### Slide: Part1. What are the challenges of mobile app?
Let’s try to see what are the ingredients that made a classical web app transform into a mobile web app
#### Slide: Size
First of all, fLet’s face it… Size does matter
- you can not display the same amount of information in that device...
- information need to be displayed differently: one columns
which us to the topic of Responsive web design
#### Slide: Responsive web design
RWD can be achieved today with  together html5+CSS3
- For ex you always have to set the view port in html to fit your device width
- you can define css depending device size, device capabilities
- don’t use pixel use percentage/em => flexible image
You can do it yourself as css expert or
#### Slide: Tools
take advantage of existing fwks.
today of on the market you have a wide choice
- jqm JS+CSS our previous scaffolding plugin used it
- twitter bootstrap mostly CSS, not mobile first
- adobe topcoat, lightweight CSS with custom build
#### Slide: Push
Today with mobile app you need to change your approach to design. Your app dont pull the server for information. App dev push contextual information to the user in the format of push notification.
In native app, there is such service:
Google Cloud Messaging for Android and 
Apple Push Notification Service for iOS
where you push short version notification
With Firefox OS web based push notification is on its way.
Your own server can send push msg through websocket or equivalent protocol.
#### Slide: Surviving offline
Next challenge : we need to survive offline
- with the mobile nature of smart phone you can not assume you’ll be connected all the time.
you walk in the street …
- deal with offline mode using caching for reading data for static data
- you also need to deal with input data 
but you may go under a tunnel, go offline 
- offline mode required architectural change in your app: no more JSP and server side rendering
So application logic/flow in server
Moved some application logic and flow on the client
You should take advantage of localStorage, indexed db when you are in offline mode all  your read/write operation
#### Slide: Synchronise
When going back online your application need to synchronize operation with the server transparently
it’s a 2 steps approach:
1. send modified data from client to server. 
2. do the actual sync: retrieve modified data from server => math sync
#### Slide: making money
we have a full mobile web app. But how to make money out of it? => How to put it on the stores?
and also how do you deal with your device capabilities not yet available in html5?
what about packaging your app for a Store?
=> Hybrid!
- phoneGap now called Apache cordova. fwk like cordova titanium will allow you to get access to your device through JS 
- the utimate goal is to cease to exist. mind the gap.
- the future firefox OS: full access to modible device through html5+JS
- develop your own plugin
#### Slide: Part II: Grails and the 3musket33rs
#### Slide: who’s behind
here the persons to blame 
sebastien->push
corinne->scaffolding generation
fabrice->cujojs
math->sync
#### Slide: Grails mobile libs and plugins
- Scaffolding plugins: last year in greach, we talked about html5-mobile-scaffolding
jqm + custom MVC + event push
- this year we have a new scaffolding plugin: lightweight css with topcoat, MVC by Spring cujo, plumbing for Pipe, Store. 
- Added features Push notification
- without prior context contextual sync
- new plugin: Grails AeroGear UP plugin (go to Sebi and corinne prez tomorrow to see it in action)
- new lib: Sync lib
#### Slide: Part III Spring cujo
#### Slide: cujojs
an arch toolkit from spring io
#### Slide quoting
The future of JS is modules not fwks
#### Slide: Modules
#### Slide: ioc
#### Slide: in action 
#### Slide: DEMO SCAFFOLDING
#### Slide: PartIV-AeroGear
#### Slide:100% mobile
AG is not a fwk
it’s rather a set of libraries
it provides all the plumbing you need for mobile
it’s not just focus on JS it supports all platforms: JS/Cordova but also native android  and iOS libs
#### Slide: web, native, hybrid
as I said why would you have to choose you can start with HTML5 and web dev
package with cordova, going hybrid, even use some of our cordova plugin to work as native
but also you can dev your client mobile with native - we’ll talk at themed on how to ease the transition -
thanks to your unified APIs
#### Slide features: Core/Push/Security/Sync
what our main features? we’re a set of libraries declined for Andoid/ios or JS
- libs for core: Pipe/Stores/auth - bare minimum for mobile app
- libs to support Push notification
- libs to Security: OTP, encryption
- future libs for sync, secure offline caching support
#### Slide Pipe
abstraction of connectivity
jax-rs compliant (work out of the box with grails rest endpoint)
but also server agnostic
Here is how you create and read a pipe in JS
similar apis exists in Java and Obj-C
#### Slide Auth
simple digest auth support with PicketLink/soon Keycloak
OAuth2 support
seamless int with pipe once authenticated
#### Slide Store

#### Slide Push notif

#### Slide Push with without UPS

#### Slide Security

#### Slide Part V Going hybrid with cordova
#### Slide What’s for
- go hybrid: present your app as a native app, by embedding a web view in your app
- cordova plugin to access your device contact etc…
- write once deploy everywhere attractive features
=> easy to write plugins for different platforms
#### Slide Cordova build
picture extracted from phone gap build (apache mbaas in the cloud)
#### Slide cordova+AG
part of our offering
aerogear-pushplugin-cordova: extend of the standard Cordova Push Plugin with AG Unified push server
aerogear-crypto-cordova: allows you to use the native aerogear crypto libs for your cordova apps. While staying close to the aerogear-js api. 
aerogear-geo-cordova:  collection of tools to make you live easier when using geo posistioning. It provides options to easly create maps that are based openlayers
aerogear-otp-cordova: Cordova plugin for OTP depends on BarcodeScanner to be able to easly obtain the secret
#### Slide the magic in 4 commands
#### Slide CORDOVA DEMO
1. Setup
BuildConfig.groovy
grails.plugin.location."aerogear-mobile-scaffolding" = "../aerogear-mobile-scaffolding"

remove all plugins excepts:
plugins {
        build ":tomcat:7.0.50.1"
        compile ":scaffolding:2.0.2"
        runtime ":hibernate:3.6.10.8"    
 }

Config.groovy
grails.scaffolding.aerogear.push.pushServerURL = "https://greach-corinnekrych.rhcloud.com"
grails.scaffolding.aerogear.push.variantID = "bbb58b3c-c5ed-4f53-8a4f-85a61e351f55"
grails.scaffolding.aerogear.push.variantSecret = "b6d6b716-410c-4e40-b39c-257398659a4e"

URLMapping: remove index

grails create-app greach

2. Scaffolding
grails cDC Talk

package totoaerogear
import grails.rest.*
@Resource(uri = '/talks', formats = ['json'])
class Talk {
    String description
    String speaker
    static constraints = {
    }
}


grails html-generate-views greachspain.Talk

cd web-app
npm install
grails run-app

3. Cordova
cordova create greach
cd greach
cordova platform add ios
cordova plugin add ../../aerogear-pushplugin-cordova
update main.js URL replace localhost twice
cp -rf ../web-app/* www
cp -rf ../web-app/config.xml .
cordova build
open xcodeproject and run
Login to UPS:
curl -3 -v -b cookies.txt -c cookies.txt -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{"loginName": "admin", "password":"admin"}' https://greach-corinnekrych.rhcloud.com/rest/auth/login

Send msg:
curl -3 -u "afded493-b9f4-41c2-a533-7b2c7e823b34:f62612a5-4a6f-4e9e-b80e-9c92e2d2132f" -v -H "Accept: application/json" -H "Content-type: application/json" -X POST -d '{ "variants" : ["bbb58b3c-c5ed-4f53-8a4f-85a61e351f55"], "message": {"key":"1", "key2":"other value", "alert":"Hola Amigo! amas conferencia Greach?"},"simple-push": "version=123"  }' https://greach-corinnekrych.rhcloud.com/rest/sender

#### Slide part VI
#### Slide Different approaches
#### Slide MATHSYNC DEMO

#### Slide Part VII Going native thanks AG
As we said unified libs
simplify switching from one platform to the other
#### Slide do not fear
Let’s compare for ex Js code we saw earlier with Pipe with obj-C one
#### Cookbook
unified=> 3 different versions#### Slide tooling

