For folks who want to mess with the new Online Services in UE5.x (aka OSSv2, aka https://docs.unrealengine.com/5.1/en-US/online-services-in-unreal-engine/), here's what I had to go to get it to work.  Might be useful for others

1) 
Install Lyra sample from UE marketplace

2) 
Edit \Unreal Projects\LyraStarterGame\Plugins\CommonUser\Source\CommonUser\CommonUser.Build.cs
Change 
```
		bool bUseOnlineSubsystemV1 = true;
```
to
```
		bool bUseOnlineSubsystemV1 = false;
```

3)
Copy everything in \Unreal Projects\LyraStarterGame\Config\Custom\EOS\DefaultEngine.ini to bottom of \Unreal Projects\LyraStarterGame\Config\DefaultEngine.ini

4) 
In \Unreal Projects\LyraStarterGame\Config\DefaultEngine.ini

Change 
```
	[OnlineSubsystemEOS]
	bEnabled=true

	[OnlineSubsystemEOSPlus]
	bEnabled=true
```
to 
```
	[OnlineSubsystemEOS]
	bEnabled=false

	[OnlineSubsystemEOSPlus]
	bEnabled=false
```
And add this line:
```
	[/Script/Engine.OnlineEngineInterface]
	bUseOnlineServicesV2=true
```
as mentioned in https://docs.unrealengine.com/5.2/en-US/common-user-plugin-in-unreal-engine-for-lyra-sample-game/

5)
Replace:
```
;For OSSv2, fill in the following lines with actual details and then uncomment
;+[OnlineServices.EOS]
;+ProductId=PRODUCTID
;+SandboxId=SANDBOXID
;+DeploymentId=DEPLOYTMENTID
;+ClientId=CLIENTID
;+ClientSecret=CLIENTSECRET
```
with your values.  

You can get real EOS values by following this tutorial:
https://dev.epicgames.com/community/learning/courses/1px/unreal-engine-the-eos-online-subsystem-oss-plugin/Lnjn/unreal-engine-introduction
which talks about OnlineSubsystem (aka OSSv1) but its the same values should work.

One major stumbling block for me is that this doesn't work:

```
;For OSSv2, fill in the following lines with actual details and then uncomment
+[OnlineServices.EOS]
+ProductId=PRODUCTID
+SandboxId=SANDBOXID
+DeploymentId=DEPLOYTMENTID
+ClientId=CLIENTID
+ClientSecret=CLIENTSECRET
```

It has to be:

```
;For OSSv2, fill in the following lines with actual details and then uncomment
[OnlineServices.EOS]
ProductId=PRODUCTID
SandboxId=SANDBOXID
DeploymentId=DEPLOYTMENTID
ClientId=CLIENTID
ClientSecret=CLIENTSECRET
```

or function that reads this material (\UnrealEngine\Engine\Plugins\Online\OnlineServicesEOSGS\Source\Private\Online\OnlineServicesEOSGSPlatformFactory.cpp) doesn't read the values and causes all the OnlineServices interfaces to return null.

You can also set:
```
EncryptionKey=ENCRTYPTIONKEY
```
here if you need it in this section as well. OnlineServicesEOSGSPlatformFactory.cpp reads it. Not sure what it does but fyi.

6)
Now generate VS projects as normal and open in Visual Studio.

You can edit 
Unreal Projects\LyraStarterGame\Plugins\CommonUser\Source\CommonUser\Private\CommonSessionSubsystem.cpp
to add OnlineServices code to UCommonSessionSubsystem::BindOnlineDelegatesOSSv2() for example:

eg:
	
```
	TSharedPtr<IOnlineServices> OnlineServices = GetServices(GetWorld());

	IAchievementsPtr AchievementsPtr = OnlineServices->GetAchievementsInterface();
	UE_LOG(LogTemp, Log, TEXT("AchievementsPtr valid:%d"), AchievementsPtr != nullptr);

	IAuthPtr AuthPtr = OnlineServices->GetAuthInterface();
	UE_LOG(LogTemp, Log, TEXT("AuthPtr valid:%d"), AuthPtr != nullptr);
```

7)
When you build you'll get build errors in 
\LyraStarterGame\Plugins\CommonUser\Source\CommonUser\Public\CommonSessionSubsystem.h

and

\LyraStarterGame\Plugins\CommonUser\Source\CommonUser\Public\CommonUserSubsystem.h

To fix, add 

```
#include "Online/OnlineAsyncOpHandle.h"
```
to both

8)
You'll get build errors that these headers don't exist:

```
#include "Interfaces/OnlineIdentityInterface.h"
#include "OnlineError.h"
```

so just comment them out.

9) 
After all this, you should get when you play in the UE5 editor you should see logs like:

```
LogTemp: AchievementsPtr valid:1
LogTemp: AuthPtr valid:1
```

I can't speak to the rest of Lyra working with OnlineServices but hopefully this has been useful.


