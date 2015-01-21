#Table of Contents
- [1. System Requirements](#user-content-1-system-requirements)
  - [a. For Development](#user-content-a-development)
  - [b. Using the Plugin in Unity Games](#user-content-b-using-the-plugin-in-unity-games)
- [2. Adding the Plugin to your Unity Project](#user-content-2-adding-the-plugin-to-your-unity-project)
  - [a. Importing](#user-content-a-import)
  - [b. Connect Package Scripts](#user-content-b-connect-package-scripts)
  - [c. Attach R1ConnectPluginCommon script to the game object](#user-content-c-attach-r1connectplugincommon-script-to-the-game)
- [3. Using Connect Scripts](#user-content-4-using-connect-scripts)
  - [a. Basic Initialization](#user-content-a-basic-initialization)
  - [b. Advertising Actions](#user-content-b-advertising-actions)
  - [ i. Ad Mediation](#user-content-i-ad-mediation)
  - [c. Analytics Actions](#user-content-c-analytics-actions)
    - [ i. Automatic Events](#user-content-i-automatic-events)
    - [ ii. Custom Events](#user-content-ii-custom-events)
    - [ iii. Common Events](#user-content-iii-common-events)
- [4. Building Your Project](#user-content-5-building-your-project)
  - [a. For iOS Targets](#user-content-a-for-ios-targets)
  - [b. For Android Targets](#user-content-b-for-android-targets)

#1. System Requirements

The RadiumOne Connect Plugin for Unity provides advertising and analytics services to Unity game developers targeting iOS and Android devices.


##a. Development

While Unity development is possible using Windows OS, if you expect to deliver an iOS-compatible application final output to iOS requires Apple's Xcode 6.x (or newer) development tool (Mac OS version 10.8 or newer).

##b. Using the Plugin in Unity Games

The Connect Plugin Package (R1ConnectPlugin.unitypackage) supports both iOS and Android devices.

### iOS

The Connect plugin supports iOS 6.0 and later.  This setting can be found in the Xcode project settings.

The following iOS architectures are supported:

* arm7
* arm7s
* arm64

While the Connect plugin supports these architectures it is not necessary for your iOS application to support all of them. 


### Android

The Connect plugin supports OS 2.3+.  Google Play Services 4.0 or later is required.

#2. Adding the Plugin to your Unity Project

##a. Import

Add the RadiumOne Connect Plugin to your Unity project by:
- clicking the 'Assets' menu in the Unity development app and moving your pointer over the 'Import Package' option to reveal
- selecting the 'Custom Package...' option
- navigating to the RadiumOne Connect Plugin package file that you previously placed on your computer and open it

Alternatively, you may navigate to the file on your computer and simply open the package (i.e. double-click, etc.) to bring up the 'Importing Package' window.  From there, you can import all the files in the package to target both iOS and Android devices or deselect either of the platform-specific file sets if you intend to deploy to one.

##b. Connect Package Scripts

The Plugin package contains the following script files:
* R1ConnectPluginCommon.cs
* R1ConnectPluginDefault.cs
* R1ConnectEngageAndroid.cs
* R1ConnectEngageiOS.cs
* R1ConnectAnalyticsAndroid.cs
* R1ConnectAnalyticsiOS.cs
* R1ConnectUserInfo.cs
* R1DictionaryStruct.cs
* R1ConnectEmitItem.cs
* R1ConnectEmitterLineItem.cs

It is required to include all scripts in your project when you import the plugin package, though if you are not targeting both iOS and Android devices for your application, it is not required to include the platform specific script files -- those with the platform name appended to the script name:
* R1ConnectEngageAndroid
* R1ConnectAnalyticsAndroid

and
* R1ConnectEngageiOS
* R1ConnectAnalyticsiOS


There are platform specific files within the 'Plugins' sub-directories 'Android' and 'iOS'.

Import the files listed in the platform directory for each supported OS on which you plan to deploy your application.  You won't need to refer to any of these files directly while coding your application in Unity; the Connect for Unity Plugin scripts will automatically utilize them when you build your project for a target platform.

The Android specific library files include:
* libR1ConnectEngage.jar - required
* libR1AdMobMediationAdapter.jar - optional...include if ad mediation is setup using the AdMob network
* libR1MoPubMediationAdapter.jar - optional...include if ad mediation is setup using the MoPub network

The iOS specific library files include:
* libR1ConnectEngage - required
* libR1AdMobMediationAdapter - optional...include if ad mediation is setup using the AdMob network
* libR1MoPubMediationAdapter - optional...include if ad mediation is setup using the MoPub network

Verify that you are building your Unity project for iOS or Android.  File > Build Settings... > Android/iOS > Switch Platform
Without specifying a supported platform, the compiler will throw an error.

##c. Attach R1ConnectPlugin script to the game object
Attach the `R1ConnectPluginCommon.cs` to the game object where you want to receive the callbacks of earned rewards, ad closed and list of completed ads.

#3. Using Connect Scripts

All Connect functions are called off the R1ConnectPluginCommon class.

##a. Basic Initialization

First, specify your applicationID.  You only to do this once in your application; it isn't desirable to have the object that has an attached script which calls Connect to repeatedly load and unload.  Therefore, it is suggested that you attach a custom script to your camera. Then, in an appropriate place in the script (e.g. the Start() call of a customized MonoBehavior class) set up the applicationID:

    R1ConnectPluginCommon.SetApplicationId("YOUR APPLICATION ID");
    //Ask your RadiumOne contact for an app id

Once the Connect Plugin has its applicationID set, enable one or both of its activity modules: Analytics and DisplayAds.

You can enable these modules with:
    R1ConnectPluginCommon.EnableAnalytics();
    R1ConnectPluginCommon.EnableDisplayAds();

As with setting the applicationID, these modules should only be enabled once per application launch.


On Android, in addition to the above steps, add the following line to your start method

    R1ConnectPluginCommon.ConnectStart();

And override OnApplicationPause of your script by adding the following line

    R1ConnectPluginCommon.ConnectStop();
    
Also attach the R1ConnectPluginCommon.cs to the main camera or any other game object that wants to listen to the callbacks from advertising flows.     


##b. Advertising Actions


For DisplayAd activities, the Connect Plugin supports full-screen views (Offerwall, Interstitial, Video) and Banners. When you invoke these methods, you have the option to supply a dictionary of placementIds.

PlacementsIds represent R1Engage TrackIds and third party adUnitIds (that are used for Ad Mediation - discussed later). An Engage TrackId is a custom string that (uniquely) identifies any given advertisement.

```
	Dictionary<string, string> placementIds = new Dictionary<string, string>();
	
	placementIds.Add (R1ConnectPluginCommon.ENGAGE_ADAPTER, Unique Engage Track Ids); // Setting a trackID allows you to later check if a user completed an offer related to an viewed advertisement, allowing you to reward them with some in app bonus or feature that you designate.
```
Pass the dictionary object to the following methods to show the full screen ads.

    R1ConnectPluginCommon.ShowOfferwall(Dictionary offerwallPlacementIds);
    R1ConnectPluginCommon.ShowInterstitial(Dictionary interstitialPlacementIds);
    R1ConnectPluginCommon.ShowVideo(Dictionary videoPlacementIds);

When you call these functions, one of three things will happen:

  1) An advertisement is found and the specific advertising view is displayed
  2) An advertisement is not available for any of the ad networks if ad mediation is enabled and nothing is displayed
  3) Some error occurred and nothing is displayed

As the Connect Plugin is intended to be an easy, lightweight method to integrate advertisements into your application, there are no status values to check or error handling to deal with in your Unity code.  The advertisements display if possible, otherwise the request will silently fail.


To display an Ad Banner, create the dictionary of placementIds for banner ad as discussed above for other ad types. You can choose to display banner from these options.


	R1ConnectPluginCommon.ShowSmartBanner(Dictionary bannerPlacementIds, R1ConnectPluginCommon.BannerPosition position)

	R1ConnectPluginCommon.ShowBanner(Dictionary bannerPlacementIds, R1ConnectPluginCommon.BannerSize type, R1ConnectPluginCommon.BannerPosition position)

'ShowSmartBanner' takes a parameter to allow you to specify if you want the banner to appear at the top or bottom of the screen:

	R1ConnectPluginCommon.BannerPosition.POSITION_TOP
	R1ConnectPluginCommon.BannerPosition.POSITION_BOTTOM


A banner that is appropriately sized for the current device screen size and game view orientation will be automatically selected.

Alternatively, 'ShowBanner' can be used to create a banner of a specific predefined size using the following constants:

	R1ConnectPluginCommon.BannerSize.BANNER_300_50 for 300x50
	R1ConnectPluginCommon.BannerSize.BANNER_320_50 for 320x50
	R1ConnectPluginCommon.BannerSize.BANNER_300_250 for 300x250
	R1ConnectPluginCommon.BannerSize.BANNER_480_50 for 480x50
	R1ConnectPluginCommon.BannerSize.BANNER_728_90 for 728x90
	R1ConnectPluginCommon.BannerSize.BANNER_1024_90 for 1024x90

'ShowBanner' requires that you specify if the banner should appear at the top or bottom of the screen using:

	R1ConnectPluginCommon.BannerPosition.POSITION_TOP
	R1ConnectPluginCommon.BannerPosition.POSITION_BOTTOM


While the Offerwall, Interstitial and Video ad types all provide a method for the user to dismiss them from view, the Banner ad type is removed from the screen using:

	R1ConnectPluginCommon.HideBanner()

Once called, the banner will be removed from the screen and all memory related to the banner released.


To respond to specific interesting events that can occur as a result of using the Connect Plugin, you can write and register callbacks that will be called every time these events occur.

The 'adWillGainFocus' callback method is invoked when a user enters an advertising flow. It takes the form:

````
	public void HandleAdWillGainFocus(){
		// do something - this is an ideal time to pause the game before an Ad appears or expands and obscures the game scene
	}
````

The 'adDidLoseFocus' callback method is invoked when a user exits an advertising flow. It takes the form:

````
	public void HandleAdDidLoseFocus(){
		// do something - this is an ideal time to un-pause the game (if you previously paused it) as the Ad disappears or collapses to reveal the game scene
	}
````


The 'didReceiveNewReward' and 'didReceiveCompletedOffers' callback methods may be invoked after an advertisement is viewed or dismissed as well as upon return of a manual call to CheckCompletion().  They should take the form similar to the following:

````

	public void HandleDidReceiveNewReward(int rewards){
		// do something with rewards - rewards value is in in-app currency
		// as set up for your application in our Engage portal online.
	}
	
	public void HandleDidReceiveCompletedOffers(string [] ids){
		// do something with array of trackIDs returned.  For example, you might
		// reward users as particular trackIDs are completed (and returned here).
		// Or you might unlock content for them after a certain amount of trackIDs
		// have been returned.  You can be as creative as you want.
	}
````

Register these handlers (for each you have written) in the onEnable method and unregister them in the onDisable method of your script.  It is important to carefully choose which object or scene in your application will implement the script for registering and handling the rewards/completed offers callbacks.  Since your application may be notified at any time by these callbacks, you want to attach your script to a game element that is fairly permanent (doesn't tend to unload and reload) because once the advertising engine attempts to notify the application of a reward or completed offer, it will not repeat the notification.  It's a one-time event so you app needs to be listening.

```
	void OnEnable()
	{
		R1ConnectPluginCommon.adWillGainFocus += HandleAdWillGainFocus;
		R1ConnectPluginCommon.adDidLoseFocus += HandleAdDidLoseFocus;
		R1ConnectPluginCommon.didReceiveNewReward += HandleDidReceiveNewReward;
		R1ConnectPluginCommon.didReceiveCompletedOffers += HandleDidReceiveCompletedOffers;
	}
	void OnDisable()
	{
		R1ConnectPluginCommon.adWillGainFocus -= HandleAdWillGainFocus; 
		R1ConnectPluginCommon.adDidLoseFocus -= HandleAdDidLoseFocus; 
		R1ConnectPluginCommon.didReceiveNewReward -= HandleDidReceiveNewReward;
		R1ConnectPluginCommon.didReceiveCompletedOffers -= HandleDidReceiveCompletedOffers;
	}
```	

CheckCompletion will send an asynchronous query to our servers to return (if applicable) rewards earned (in your app's currency) and the list of trackIDs associated with completed offers - if no offers have been completed then no trackIDs will be returned.
TrackIDs are values specified by your application to (uniquely) identify any given advertisement.  You set a trackID prior to displaying an advertisement (explained below in additional configuration options).
Normally, it is not necessary to call CheckCompletion manually as its checks are typically done automatically as advertisements are viewed and dismissed.  But you may choose to call this upon app launch to ensure there are no completed items waiting to be gathered.


````
	R1ConnectPluginCommon.CheckCompletion()
````

### i. Ad Mediation

Ad mediation allow your application to pull from a wider pool of ad networks to fulfill an ad request giving you confidence that there will be ads to display when you want them.  Engage supports mediation by enabling ad fulfillment via the following ad networks:

* Google AdMob - https://www.google.com/ads/admob/
* MoPub - http://www.mopub.com

#### Enabling ad mediation

To enable mediation in your application is a four step process.

1) Set up an account and create placement ids (adUnits) in the AdMob, MoPub web portals.  AdUnitIds are tied to each ad you place in your application and are unique for each platform to which you deploy.  You should create a unique AdUnitId for each ad placement in your application for each platform to which you intend to deploy.

2) Add the mediation adapter libraries (of each desired network) to your project.  This was discussed in the section  - [b. Connect Package Scripts](#user-content-b-connect-package-scripts)

For Android targets include the MediationAdapter.jar file for any network that you've activated in the previous step:

* libR1AdMobMediationAdapter.jar
* libR1MoPubMediationAdapter.jar

For iOS targets, the MediationAdapter libraries are:

* libR1AdMobMediationAdapter.a
* libR1MoPubMediationAdapter.a


3) In your code, enable mediation behavior with the following line of code:
```
    R1ConnectPluginCommon.EnableAdMediation();
```

It is recommended that you make this call at the same place where you enable displayAds:
```
    R1ConnectPluginCommon.EnableAdMediation();
    R1ConnectPluginCommon.EnableDisplayAds();
```

4) The adUnit ids that you generate in the web portals must be used to initialize each advertisement you want to display in your application.  This can be accomplished by adding the appropriate adUnitId for each activated mediation network to the placementIds dictionary that you can pass to each advertisement you display in your application. Note that AdMob and MoPub both specify unique keys between platforms so you will need to generate and apply the unique AdUnitIds for each supported platform.
```
	Dictionary<string, string> bannerPlacementIDs = new Dictionary<string, string>();
	
	bannerPlacementIDs.Add (R1ConnectPluginCommon.ENGAGE_ADAPTER, Unique Engage Track Ids); // Setting a trackID allows you to later check if a user completed an offer related to an viewed advertisement, allowing you to reward them with some in app bonus or feature that you designate. If Ad mediation is not turned off then simply put the track id for Engage
    #if UNITY_IPHONE
	bannerPlacementIDs.Add(R1ConnectPluginCommon.ADMOB_ADAPTER, Your iOS AdMob placement key for banner goes here);
	bannerPlacementIDs.Add (R1ConnectPluginCommon.MOPUB_ADAPTER, Your iOS MoPub placement key for banner goes here);
    #elif UNITY_ANDROID
	bannerPlacementIDs.Add(R1ConnectPluginCommon.ADMOB_ADAPTER, Your Android AdMob placement key for banner goes here);
	bannerPlacementIDs.Add (R1ConnectPluginCommon.MOPUB_ADAPTER, Your Android MoPub placement key for banner goes here);
	#endif
	R1ConnectPluginCommon.ShowSmartBanner(bannerPlacementIDs, R1ConnectPluginCommon.BannerPosition.POSITION_TOP);
```

That's all that is required in your app! It is required to also configure the rules by which Engage will execute mediation.  The mediation rules can be managed via the RadiumOne Engage web portal (http://gwallet.net/gwallet-admin).  Now, whenever, you request an advertisement, the Connect Unity Plugin will automatically attempt to fill the request according to the rules you specified in the RadiumOne Engage web portal.


###1. Additional Configuration Options

There are several optional values you can set that will add to the quality of your analytics or advertising.

    R1ConnectPluginCommon.SetUserId(string userId)
	// An identifier you determine that for the current user (if you don't use this function we’ll
	// use the appropriate platform Advertising Identifier in its place).

    R1ConnectPluginCommon.SetGender(R1ConnectPluginCommon.UserGender gender)
	// The gender of your user. Use one of the following predefined valued:
    R1ConnectPluginCommon.UserGender.GENDER_UNSPECIFIED
    R1ConnectPluginCommon.UserGender.GENDER_MALE
    R1ConnectPluginCommon.UserGender.GENDER_FEMALE
	
    R1ConnectPluginCommon.SetAge(int age)
	// The age of your user.
	

##c. Analytics Actions

The Analytics module of the Connect Plugin enables, automatic, custom and common events in order to get the most meaningful data on how users interact with your app.

### i. Automatic Events

These events are triggered when the state of the application is changed, and therefore do not require any additional code on your part.

**Launch** - emitted when the app starts

**First Launch** - emitted when the app starts for the first time

**First Launch After Update** - emitted when the app starts for the first time after a version upgrade

**Suspend** - emitted when the app is put into the background state

**Resume** - emitted when the app returns from the background state

**Application Opened** - emitted when the app is opened and can measure when your app is opened after a message is sent

**Session Start** - emitted when a new session begins

**Session End** - emitted when a session ends; includes a Session Length attribute with the session length in seconds.


###ii. Custom Events

While the Connect Plugin provides several automatic and predefined events, you may want to add analytic markers that are more closely aligned with your application.  In that case, you will want to utilize the custom event.

    R1ConnectPluginCommon.EmitCustomEvent(string customEvent, Dictionary<string,string> param) // Takes an event name and dictionary of key-value pairs that can further define the event.

If planned and structured correctly, custom events can be strong indicators of user intent and behavior. Some examples include pressing the “like” button, playing a song, changing the screen mode from portrait to landscape, and zooming in or out of an image. These are all actions by the user that could be tracked with events.

**Best Practices for Custom Events**

One common mistake is to parametrize event names (with user data for example). Event names should be hard-coded values that you use to segment data on a specific category of event. 

Example: "ProfileViewing"

Avoid: "Profile Viewing - Lady Gaga's profile"

As you may have thousands of user profiles in your database, it is preferable to keep the event name high level ("ProfileViewing") so you can run interesting analytics on it. High level events help answer questions like "how many profiles does a user visit every day on average?" 

Another common mistake is to add parameters to the event that have too many possible values. To follow up on the previous example, one may decide to add the number of profile followers as an event parameter:
			  			   
Again, the problem here is that each profile may have any number of followers. This will fragment your data too much to extract any valuable information.

A good strategy would be to define relevant buckets for high variance parameters. In this case, it might be more relevant to separate traffic on the profiles with a high follower count from traffic on profiles with a low count. You could define 3 categories: 

- "VERY_INFLUENTIAL" for profiles > 100,000 
- "INFLUENTIAL" for profile > 10,000 and <= 100,000
- "NON_INFLUENTIAL" for profile <= 10,000

###iii. Common Events

The Connect Plugin provides a list of common events that cover all the typical user flows (login, registration, share, purchase, etc) in a standardized format for optimized reporting in the RadiumOne Connect portal, providing a great foundation to your analytics strategy. Once set up in your code, they unlock great insights, particularly on user lifetime value.

*Note: The last argument in all of the following emitter callbacks, 'param', is a dictionary of “key”,”value” pairs or nil, which enables you to customize these events as much as you want.*

    R1ConnectPluginCommon.EmitUserInfo(R1ConnectUserInfo userInfo, Dictionary<string,string> param) // Enables you to capture user profiles
    R1ConnectPluginCommon.EmitLogin(string userId, string userName, Dictionary<string,string> param) // Tracks a user login within the app
    R1ConnectPluginCommon.EmitRegistration(string userId, string userName, string country, string city, string state, Dictionary<string,string> param) // Records a user registration within the app
    R1ConnectPluginCommon.EmitFBConnect(string[] socialPermissions, Dictionary<string,string> param) // Allows access to Facebook services
    R1ConnectPluginCommon.EmitTConnect(string userId, string userName, string[] socialPermissions, Dictionary<string,string> param) // Allows access to Twitter services
    R1ConnectPluginCommon.EmitUpgrade(Dictionary<string,string> param) // Tracks an application version upgrade
    R1ConnectPluginCommon.EmitTrialUpgrade(Dictionary<string,string> param) // Tracks an application upgrade from a trial version to a full version
    R1ConnectPluginCommon.EmitTransaction(R1ConnectEmitItem purchase, Dictionary<string,string> param) // Tracks the action of the user completing a purchase in your app
    R1ConnectPluginCommon.EmitTransactionItem(string transcationId, R1ConnectEmitterLineItem item, Dictionary<string,string> param) // Tracks the details of a purchase associated with a transaction
    R1ConnectPluginCommon.EmitCartCreate(string cartId, Dictionary<string,string> param) // Tracks the action that the user has initiated the intent to purchase something from within your app
    R1ConnectPluginCommon.EmitCartDelete(string cartId, Dictionary<string,string> param) // Tracks the action that the user has left or otherwise stopped being in a mode to purchase items within your app
    R1ConnectPluginCommon.EmitAddToCart(string cartId, R1ConnectEmitterLineItem item, Dictionary<string,string> param) // Tracks a users action of indicating interest in an item for potential purchase
    R1ConnectPluginCommon.EmitRemoveFromCart(string cartId, R1ConnectEmitterLineItem item, Dictionary<string,string> param) // Tracks a users action of declining interest in an item for potential purchase


#4. Building Your Project

As you build and run your Unity project targeting iOS and Android targets, you will need to, at least initially, set up any project level settings required to support the RadiumOne Connect Plugin.

##a. For iOS Targets

As noted earlier, the Connect Plugin supports devices running iOS 6.0 or newer.  Therefore, in your Xcode project, under the 'General' project settings tab you must set 'Deployment Target' to be 6.0 or higher.

The only additional requirement for building your project for iOS to support the Connect Plugin is to link against the needed set of system frameworks.

Again, in Xcode, navigate to the "Build Phases" project settings and add the following frameworks (if they are not already present):

* AdSupport.framework - marked Optional instead of Required
* CoreBluetooth.framework
* CoreLocation.framework
* CoreTelephony.framework
* Security.framework
* StoreKit.framework
* SystemConfiguration.framework
* WebKit.framework - marked Optional instead of Required
* libsqlite3.dylib

It is important to add an entry to the 'Other Linker Flags' setting in your application's Build Settings in Xcode. Add '-ObjC' to the 'Other Linker Flags' setting if it is not already present. This is a common required flag when integrating static libraries with your code.

**Additional Setup For Ad Mediation**
If your Unity application is using the advertising feature of the SDK and additionally has enabled ad mediation using MoPub, there is a short list of resource files that you must manually add to your Xcode project for MoPub advertisements to display correctly.

Alongside the R1UnityConnect.unitypackage file, you should find an 'iOS Resources' folder.  Within that folder is the 'MoPubResources' folder which contains:

* MPCloseButtonX.png
* MPCloseButtonX@2x.png
* MPCloseButtonX@3x.png
* MRAID.bundle
* MPAdBrowserController.xib

These files are all required to be included in your application for MoPub advertisements to be displayed correctly.  Again, these files are only required if you are using the advertising component of R1UnityConnect and have mediation enabled and MoPub setup.  Once the Xcode project has been created by building in Unity, you can add the three png image files to your Xcode project's Images.xcassets catalog.  The MRAID.bundle and MPAdBrowserController.xib can be placed with any of the other auxiliary resource files in the Xcode project. e.g. application plist.

**Note:**
As is standard for Unity projects, if you Replace the Xcode project when you 'Build and Run' from within the Unity development app, you will need to re-apply these customizations to your Xcode project.  After the initial setup of the Xcode project, it is suggested that you save your project to the same file for every build.  When the Unity app warns that an existing project file already exists, use the 'Append' option so that your code changes are applied but your project settings are left intact.

##b. For Android Targets

Add the latest google-play-services.jar file into Assets/Plugins/Android directory and update the version of google play services in AndroidManifest.xml as shown below:

```java
 <meta-data android:name="com.google.android.gms.version"
        android:value="5077000" />
```        

**Additional Configuration for Mediation Support**

**Adding AdMob SDK and its adapter**

Make sure you have your account setup on the AdMob website and have created all the ad placements required by your application, as described above. AdMob SDK is supported by Google play services so it does not require you to download the SDK. Add the proper version of Google play services in build.gradle as explained above.

Include the google-play-services_lib as an Android library to the plugin project. You can find google-play-service.jar file in ```<ANDROID_SDK_DIR>/extras/google/google_play_services/libproject/google-play-services_lib/libs```

Add the Ad view Activity in the AndroidManifest.xml.
```java
<activity android:name="com.google.android.gms.ads.AdActivity"
       android:configChanges="keyboard|keyboardHidden|orientation|screenLayout|uiMode|screenSize|smallestScreenSize"
       android:theme="@android:style/Theme.Translucent" />
```

Confirm the correct version of google play services in manifest file.

```java
<meta-data android:name="com.google.android.gms.version"
        android:value="XXXXXXX" />
```        

**Adding MoPub SDK and its adapter**

Make sure you have your MoPub account setup on the MoPub website and have created all the ad placements required by your application, as described above. The MoPub SDK and integration documentation is available here:

https://github.com/mopub/mopub-unity-android-plugin

From the MoPub documentation, create the MoPubPlugin.jar file and copy it to Unity project's Plugins/Android directory. Also include the libR1MoPubMediationAdapter.jar file if you have not included while importing unity package.

Add the MoPub specific activities in the AndroidManifest.xml file
```java
<activity android:name="com.mopub.mobileads.MoPubActivity" android:configChanges="keyboardHidden|orientation"/>
<activity android:name="com.mopub.mobileads.MraidActivity" android:configChanges="keyboardHidden|orientation"/>
<activity android:name="com.mopub.mobileads.MraidBrowser" android:configChanges="keyboardHidden|orientation"/>
<activity android:name="com.millennialmedia.android.MMActivity"android:theme="@android:style/Theme.Translucent.NoTitleBar" android:configChanges="keyboardHidden|orientation"/>
<activity android:name="com.millennialmedia.android.VideoPlayer"android:theme="@android:style/Theme.NoTitleBar.Fullscreen" android:configChanges="keyboardHidden|orientation|keyboard"/>
<activity android:name="com.mopub.common.MoPubBrowser" android:configChanges="keyboardHidden|orientation"/>
<activity android:name="com.mopub.mobileads.MraidVideoPlayerActivity" android:configChanges="keyboardHidden|orientation"/>
```

Enjoy!
