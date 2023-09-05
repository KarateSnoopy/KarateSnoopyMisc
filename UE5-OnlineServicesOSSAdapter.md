For folks who want to mess around with the new Online Services (aka OSSv2, aka https://docs.unrealengine.com/5.1/en-US/online-services-in-unreal-engine/) while using the OSS Adapter plugin (OnlineServicesOSSAdapter), I didn't find anyone talking about it.  So I debugged through the code and figured out how to get it working.  Might be useful for others

Also see my other guide https://github.com/KarateSnoopy/KarateSnoopyMisc/blob/main/UE5-OnlineServices.md for how to setup Online Services in general.

1) 
Setup DefaultEngine.ini and Build.cs the same way you'd setup to use the OnlineSubsystem (aka OSSv1).
There's plenty of examples and tutorials of how to do this for the OnlineSubsystem plugins so I won't cover that.

2)
In .uproject file, add these plugins if missing.  
This example uses OnlineSubsystemSteam but it can be any of the OSS plugins.

```
		{
			"Name": "OnlineServices",
			"Enabled": true
		},
		{
			"Name": "OnlineServicesOSSAdapter",
			"Enabled": true
		},
		{
			"Name": "OnlineSubsystemSteam",
			"Enabled": true
		}
```

3) 
Add OnlineServicesInterface and OnlineSubsystemUtils in Build.cs (as seen in Lyra sample)

```
		PublicDependencyModuleNames.Add("OnlineServicesInterface");
		PrivateDependencyModuleNames.Add("OnlineSubsystemUtils");
```
		
4) 
In DefaultEngine.ini, add something like this.
This part I couldn't find any docs on, but I debugged through the code and it seems to work.
Here's an example for adding Steam:

```
	[/Script/Engine.OnlineEngineInterface]
	bUseOnlineServicesV2=true

	[OnlineServices.OSSAdapter]
	+Services=(Service="Steam",ConfigName="OSSAdapter.Steam",OnlineSubsystem="Steam")

	[OnlineServices]
	DefaultServices=Steam

	[OnlineSubsystem]
	NativePlatformService=Steam
```

Its not clear what ConfigName is used for but it seems like it can be anything.

5)
Copy the C++ code that uses OnlineServices to one of the classes in your map.
You can use Lyra as an example.
eg:

```
	DefaultContextInternal->OnlineServices = GetServices(EOnlineServices::Default, NAME_None);
	check(DefaultContextInternal->OnlineServices);

	IAchievementsPtr AchievementsPtr = DefaultContextInternal->OnlineServices->GetAchievementsInterface();
	UE_LOG(LogTemp, Log, TEXT("AchievementsPtr valid:%d"), AchievementsPtr != nullptr);
```

6)
When you build & launch, and launch the game as standalone you should see the Steam signin UI pop.
