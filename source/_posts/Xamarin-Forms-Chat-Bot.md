---
title: Xamarin Forms Chat Bot
date: 2018-01-26 18:11:03
tags:
---

# Xamarin Forms Chat Bot

[Xamarin Dev Days in Cape Town](https://ti.to/xamarin/dev-days-cape-town-2017) introduced me to [Xamarin Forms](https://www.xamarin.com/forms), which basically allows you to share C# code accross native Android & iOS apps.  

This app was targeted towards speech recognition. Having my own Android device and coding on Windows, I could really only build and test on Android. However, still worth keeping in mind that eventually this will need to result in an iOS app as well, so all NuGet packages and libraries will have to be usable in both projects.  

---
## App Design
Getting the hang of which elements are usuable where/when was really difficult at first, but with lots of research online I started getting used to it. Being comfortable with SPA two-way binding frameworks, the [`INotifyPropertyChanged`](https://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.aspx) implementation seemed very cumbersome on every page for all bindable properties.  
After some digging I found [Fody PropertyChanged](https://github.com/Fody/PropertyChanged) which did all the heavy lifting for me! It really was as simple as installing the NuGet package, adding a new `FodyWeavers.xml` file to both iOS & Android projects, then finally just implementing the `INotifyPropertyChanged` on my VM with its single property:  
```cs
public class MasterViewModel : INotifyPropertyChanged
{
  public event PropertyChangedEventHandler PropertyChanged;
}
```

---
## Speech-to-Text & Text-to-Speech
After the pages were ready, next up was the implementation of _speech-to-text (stt)_ and potentially _text-to-speech (tts)_. Started off by trying some [Xamarin Forms Components](https://components.xamarin.com/) in the hopes to share all its code as well. Some of it worked, but often fell short in certain areas or mobile versions. I think it's very difficult to keep those components 100% compatible with all platforms, especially with the different OS APIs continuously evolving as well.  

Keeping the domain logic in the shared project, I needed a way of triggering the native activities and retrieving its results. [Xamarin Messaging Center](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/messaging-center/) is a pub/sub service which allows to fire of events without needing to know where it will be executed. This worked well, but eventually became really messy keeping track of multiple subscriptions and difficult to debug at times.  

The final refactor of this was to remove the messaging center and use the [Xamarin Dependency Service](https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/dependency-service/). In order to create a better _conversation effect_ between _tts_ & _stt_ in Android, the events could be queued with the help of Android's [Utterance Progress Listener](https://developer.xamarin.com/api/type/Android.Speech.Tts.UtteranceProgressListener/).  

---
## [Dialogflow](https://dialogflow.com/)
Formerly known as API.AI, Dialogflow is a brilliant free artificial intelligence service for converting text into context. You can train your own agent and integrate it with multiple existing systems or, as in this case, with custom applications.  

There is actually an [API.AI Xamarin Component](https://components.xamarin.com/view/ApiAiSDK) which worked pretty well, but since [Google acquired API.AI](https://techcrunch.com/2016/09/19/google-acquires-api-ai-a-company-helping-developers-build-bots-that-arent-awful-to-talk-to/) and changed it to Dialogflow, a new version for Dialogflow V2 API has been released. At the time of writing (2018/01/26), no [Dialogflow SDKs](https://dialogflow.com/docs/sdks) exist for Xamarin with API V2, which to be fair, is still in BETA.   
The main difference was the authentication.
  - V1 authenticates with an access token
  - V2 authenticates with Google

With the V1 access token you could easily authenticate from the mobile app using the API.AI library. The authentication with V2 requires a bit more configuration. There are various ways to implement the [Google OAuth 2.0](https://developers.google.com/identity/protocols/OAuth2), however for my purpose I wanted to authenticate to Dialogflow on behalf of the user, NOT using the user's own credentials. This meant that I had to continue with the [OAuth 2.0 Service Account](https://developers.google.com/identity/protocols/OAuth2ServiceAccount) implementation. That involved generating a certificate, so I ended up creating a stand alone Web API.  

---
## Web API
I went with a lightweight ASPNet Core 2 Web API, which only consisted of 1 endpoint (`IntentsController`) to facilitate the communication between mobile and Dialogflow.  

Following the [Google service account .Net OAuth 2.0](https://developers.google.com/api-client-library/dotnet/guide/aaa_oauth#service-account)  guide, the implementation was pretty straight forward.  

In order to communicate with this API, I created a new `.Net Standard 2.0` project that utilized the basic [.Net HttpClient](https://msdn.microsoft.com/en-us/library/system.net.http.httpclient(v=vs.118).aspx). It didn't have to be in a new project, but I thought it best for reusability & testability, and also a small awesome experiment to see how easy it was to reference my own .Net Standard project. Turns out it was straight forward as expected, and everything just worked together nicely.