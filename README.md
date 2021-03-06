# PlayFab Custom Game Server
Out of the Box Game Server for PlayFab.com


#### Dependencies
* StrangeIoC For Unity (included)
* PlayFab Game Server SDK (included)
* PlayFab Supports Building to Windows Platform only for Game Servers 

#### Prerequisites 
* General Understanding of Unity
* Understanding of C#
* Understanding of StrangeIoC (not required, but a bonus)
* Understanding of Dependency Injection (not required, but a bonus)

#### Overview
 This repository contains the source code for a Customizable Game Server made with Unity3D.  It runs in server mode (headless) and its purpose is to provide server authoritative logic and / or Multiplayer networking functionality that can be uploaded to the PlayFab Game Servers tab.  This is an Out of the Box solution, that is super easy to customize.
 
#### Features
* Accepts and stores configuration params that are passed into the commandline of the server
* Sets up Unity Networking and waits for incoming connections
* Manages connections and disconnections
* Provides PlayFab Session authorization
* Logger
* Local Debug Mode
* PlayStream Event Subscriptions

#### Known Issues
NONE
	
	  


#### Getting Started Guide

When getting started with this custom game server it is good to know or at least understand what StrangeIoC is all about.  So I advise you to visit http://strangeioc.wordpress.com for more information about this framework.  However, it is not required to know as I'll explain the basics that you need to know to modify the game server.

StrangeIoC has an MVC (ish) style to it.  There are Views, and Controllers.  Controllers are generally called Mediators.  Since this is a game server there won't be any UI to it,  thus the views do not do anything other than provide a binding to the Mediators.  Mediators is where you will do most or all of your logic.

Once you've downloaded the project and opened up the code editor (whether it be Monodevelop or Visual Studio) you'll see some folders and I want to take a moment to review those with you.

* **StrangeIoC** - This folder contains the current version of StrangeIoC (which is open source and free).

* **PlayFabSDK** - This is a version of our SDK that was generated for use in this project.

* **Packages** - This is where you will build your modules and functionality.  I will cover packages and how to make them later in this document.  Every subfolder in Packages except for DefaultPackages is some form of an example.
 
* **GameServerContext** - This has been updates,  it use to be called MainGameServerContext and use to be the main context.  StrangeIoC has this notion of contexts which act as modules. We now call these packages.  This MainContext now auto-detects all packages and therefore you should not need to directly modify this.   
 
* **packages/DefaultPackages/PlayFabServer** - This context has one functionality and one only,  it parses the command line arguments sent in from the PlayFab Instance Generator and stores them for use in ServerSettingsData.  Which you can Inject into your mediators and have instant access to it.

* **packages/DefaultPackages/UnityNetworkingContext** - This context sets up a Unity Networking Server and listens on the port that is passed into the PlayFab Server when the instance is generated.  This all happens at startup of the server.  It then stores the NetworkManager & the NetworkClient along with other important information about who is connected to the server.  It stores all its data in UnityNetworkingData object that you also Inject into your mediators.  We will get more into the details of this later.  

#### A quick setup step before I forget to tell you!
Because the server needs to authenticate your users, it needs to know what Title to do this against. The server won't know this so you will need to set this manually.  To do this, open up the StartUp scene in the StartUpScene folder.  This is the main scene for the application and the only scene needed.  There are some advanced features of unity networking, which we won't cover in this documentation that allows scene switching and sync that to all clients. But for now let's keep things simple.   

Now that you have the scene open,  Locate the GameServerContext and expand it down.  Then Click on PlayFabServerManager.  You'll notice in the inspector that there is a ServerSettingsData component.  Put your TitleId in the Title Id meta field.

Now your ready to proceed! Don't forget to save your scene!

##### Creating a package
Packages are a great way to modularize your code and make it reusable.  While this is a great feature, it also serves as a simple way to setup your custom code and get the game server working for you. When you setup a package,  the GameServerContext auto-detects these packages in the Hierarchy and loads them in the order that it finds them.

**Example:**

- GameServerContext
	- PlayFabServerManager
	- UnityNetworkingContext
	- ChatServerManager

in the above example hierarchy all gameobject children of GameServerContext load in the order listed,  whereas PlayFabServerManager loads first and ChatServerManager loads last.  You can change the load order by changing the order in the hierarchy.

Creating a package is simple,  create a folder in the Packages folder,  and create a C# class file that extends StrangePackage.  You then have the option to load your mappings and bindings in this file and they will get auto-loaded at startup. 

**Example:**

1. Create folder named MySample in "Packages"
2. Create a file called MySampleManager.cs
3. Declare the following methods.

```

    public override void MapBindings(ICommandBinder commandBinder, ICrossContextInjectionBinder injectionBinder,
        IMediationBinder mediationBinder)
    {
    
    }

    public override void PostBindings(ICommandBinder commandBinder, ICrossContextInjectionBinder injectionBinder,
        IMediationBinder mediationBinder)
    {
        
    }

    public override void Launch(ICrossContextInjectionBinder injectionBinder)
    {
        
    }

```

In the example above we have declared a MapBindings, PostBindings and a Launch.   These do exactly as their names suggest.  You would bind your views to your mediators, signals to commands and inject data objects in MapBindings.  PostBindings is used when you have dependancies and you need something loaded prior to creating a binding.  So all MapBindings will happen first, and then PostBindings happens.  Finally, we have Launch, which allows you to perform startup logic for your package.  all three of these methods happen in the order they appear.   MapBindings --> PostBindings --> Launch

Next create a GameObject under the GameServerContext and call it MySampleManager.  Add the MySampleManager script to it. Now it will be auto-detected.

One thing to note about launch is that our GameServerContext has a signal that you can subscribe to that notifies all modules that the server has completely loaded.  From launch you can bind directly to it.  All launch methods are called prior to this signal event firing. 

##### New Mediator and View
To create a new Mediator you will need to do a few steps. 
* Create a file in your packages\MySample\Mediators folder (or create if does not exist) and name it NewExampleMediator.cs,  edit the file and extend it from the Mediator class. Then override the OnRegister() method.
 
```
public class NewExampleMediator : Mediator {
    public override void OnRegister()
    {
      
    }
}
```

* You also want to create a View file in the Views folder.  Not much goes into the view unless you want to put metadata that you would like to reference from inside the editor. (handy for debugging *winks*)

```
public class NewExampleView : View
{
    //you don't need this, i was just showing you that you can reference this.
    public float CheckAfterSeconds; 
}
```
* In your mediator you can reference the view by injecting it, like this.
```
public class NewExampleMediator : Mediator {
    [Inject] public NewExampleview View {get; set;}
    public override void OnRegister()
    {
       Debug.Log(View.CheckAfterSeconds);
    }
}

```
* Now you must register the mediator and view in the StrangePackage manager (MySample/MySampleManager.cs).    To register a binding, add the following to the MapBindings method of your PackageManager.
```
mediationBinder.Bind<NewExampleView>().To<NewExampleMediator>();
```
*  There is only one last step and then your ready to put your custom code in.  You will need to create a GameObject in the Scene, (I know Right! Seems a little weird) However it's not, the scene is running because it is unity. But since the server is running in headless mode, you can't really see it unless!!! Your debugging your server locally (handy!) I'll get more into that later.  Once you created a GameObject in the Scene,  you want to add the View to the object.  So if you know how to add components in Unity then this should be pretty easy for you. If not, select your gameobject click on the add component button and add  NewExampleview to it.  Now it's probably a good idea to rename the GameObject to  NewExampleView  or something like that.
  
That's it!  you are ready to put your custom code in!  If you run the project in the Editor, you should see the CheckAfterSeconds output to the console.

That is the basics of how the Mediators & Views work.

#### Using PlayFab
Using PlayFab is quick and easy.  There is no special work that you need to do other than specify "Using PlayFab;" and "Using PlayFab.ServerModels" in your mediators.
For example if you wanted to get TitleData from within a mediator it might look something like this.

```
using PlayFab;
using PlayFab.ServerModels;


public void GetTitleData(){
    PlayFabServerAPI.GetTitleData(new GetTitleDataRequest(), OnGetTitleData, OnPlayFabError);
}

public void OnGetTitleData(GetTitleDataResult result){
    var dataKey = result.Data["MyTitleDataKey"];
}

```


#### Using Unity Networking to send to clients.
There is a lot of documentation on how the Unity Networking client works,  so I'll let you read up there because they tell that story the best.  But I'll give you a bit of code that makes it super easy.

First thing is first, whenever want to send to a client from a mediator, you will need to inject the UnityNetworkingData object.
```
    [Inject] public UnityNetworkingData UnityNetworkingData { get; set; }
```
This object contains a property called Connections which contains a list of all connected clients to this game server instance.  This is very handy for being able to quickly find players and send messages to them.

If we are going to send a message to all players, you can iterate over the connected clients and send a message to each of them like this.

```
   foreach (var uconn in UnityNetworkingData.Connections)
   {
      uconn.Connection.Send(1000, new StringMessage()
      {
        Value="Hello everyone!"
      });        
   }
```
The first param of  .Send is the message identifier or message type of the message.  You can use one of the built in messages using MsgType.[...]  or you can make up a custom one.  But it is important to know that if you specify 1000 like I did then the client must listen for message type 1000

Here are some resources on that topic:
http://docs.unity3d.com/ScriptReference/Networking.MsgType.html
http://docs.unity3d.com/ScriptReference/Networking.NetworkConnection.Send.html

however let's say you had the users Playfab Id because they were in your friends list, and you just wanted to send that single person a message.  This is easily done like this.

```
 var FriendsPlayFabId = "12345";
 var MessageToSendToFriend = "Hey dude, come play a game with me!";
 var uconn = UnityNetworkingData.Connections.Find(c=>c.PlayFabId == FriendsPlayFabId);
 if(uconn != null){
   uconn.Connection.Send(1001, new StringMessage(){
     Value=MessageToSendToFriend
   });
 }
```

#### PlayStream Event Subscription
You can now connect to the playstream events that happen for your title.  The list of available events are listed on our [documentation site](https://api.playfab.com/playstream/docs/PlayStreamEventModels)

We have already setup an example of subscribing to playstream events.  You can either modify our example or start new.
If starting new, you will want to create a new StrangePackage and create a view and mediator.  In the mediator's register method you need to call

```
PlayFabPlayStreamAPI.Start();
```
To subscribe to all events you would do the following

```
PlayFabPlayStreamAPI.OnPlayStreamEvent += OnPlayStreamEvent
```

Then you would want to handle what happens upon each event type.
```
private void OnPlayStreamEvent(PlayStreamNotification notif)
{
	Debug.Log("received event, entity type is " + notif.EntityType);
	if (notif.EntityType != "title")
        {
        	//this is a player/character-specific event
                OnPlayerEventHappened(notif);
	}
        else
        {
        	//this is a title-specific event, could broadcast
                Debug.Log("about to send some title events");
                OnTitleEventHappened(notif);
	}        
}
```
You can also subscribe to when you are connected to the PlayStream service endpoint and when you are disconnected or if there was some sort of an error.

```
        PlayFabPlayStreamAPI.OnSubscribed += () =>
        {
            Debug.Log("connected to playstream");
        };
        PlayFabPlayStreamAPI.OnFailed += error =>
        {
            Debug.Log(error.Message);
        };
        PlayFabPlayStreamAPI.OnDisconnected += () =>
        {
            Debug.Log("Disconnected");
        };
        PlayFabPlayStreamAPI.OnError += Debug.LogException;
        
```
Once you have a playstream event notification, you can then send those to the client via Unity Networking Messages.

for example this code handles a title event and only passes it to the client if it is a TitleStatisticVersionChanged event: 
```
    private void OnTitleEventHappened(PlayStreamNotification notif)
    {
	if (notif.EventName != "title_statistic_version_changed") return;
        foreach (var conn in NetworkingData.Connections)
        {
            conn.Connection.Send(PlayStreamMsgTypes.OnPlayStreamEventReceived, new PlayStreamEventMessage() { EntityType = notif.EntityType, EventData = notif.EventObject.EventData.ToString(), EventName = notif.EventName, EventNamespace = notif.EventNamespace });
        }
    }

```

We do advise that you filter the events on the server a bit more to minimize the amount of traffic that goes to your client.

That is the basics of how to use and subscribe to PlayStream Events on the server, and use those events to send messaging to the client.

### Uploading a build and Logfiles for Unity Game Servers
This last topic is for when you upload the server to PlayFab.  
In [Game Manager](https://developer.playfab.com) under the Multiplayer --> Builds Tab, this is where you upload your build.  First you would build your game server in unity as a Win or Win64 build.  Zip the gameserver.exe & gameserver_data folder, and you would upload your build on this tab.  Once you have uploaded the build, click on the Build ID that you specified for your build.  Here you will find several settings for your build.  

For the logs to function properly, meaning that they show up in the Game Manager, you will need to append the following to the Command-line arguments.  This tells the game server where to send the Unity Log file.

```
-logFile <log_file_path>
```

If set correctly, you should see these logs under the Multiplayer --> Archived Games --> Logs (column).

 
#### Conclusion

Well I think you now have the basics of how the custom game server works.  This document gave you the barebone basics of working with the game server, Getting data from the PlayFab Server API and and how to overall get started with the Custom Game Server.  Keep an eye out on the documentation site for new tutorials on advanced topics realated to the game server at  https://api.playfab.com  and if you have questions please feel free to post in our forums at  https://api.playfab.com/community

Also be sure to check out the Client sample and the sample mediators that is included in the game server.







