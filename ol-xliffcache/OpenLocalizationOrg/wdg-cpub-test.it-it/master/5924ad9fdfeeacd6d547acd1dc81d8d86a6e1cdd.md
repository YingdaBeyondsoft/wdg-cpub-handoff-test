---
title: Register a background task
description: Learn how to create a function that can be re-used to safely register most background tasks.
ms.assetid: 8B1CADC5-F630-48B8-B3CE-5AB62E3DFB0D
---

# Register a background task


\[ Updated for UWP apps on Windows 10. For Windows 8.x articles, see the [archive](http://go.microsoft.com/fwlink/p/?linkid=619132) \]


**Important APIs**

-   [**BackgroundTaskRegistration class**](https://msdn.microsoft.com/library/windows/apps/br224786)
-   [**BackgroundTaskBuilder class**](https://msdn.microsoft.com/library/windows/apps/br224768)
-   [**SystemCondition class**](https://msdn.microsoft.com/library/windows/apps/br224834)

Learn how to create a function that can be re-used to safely register most background tasks.

This topic assumes that you already have a background task that needs to be registered. (See [Create and register a background task](create-and-register-a-background-task.md) for information about how to write a background task).

This topic walks through a utility function that registers background tasks. This utility function checks for existing registrations first before registering the task multiple times to avoid problems with multiple registrations, and it can apply a system condition to the background task. The walkthrough includes a complete, working example of this utility function.

**Note**  

Universal Windows apps must call [**RequestAccessAsync**](https://msdn.microsoft.com/library/windows/apps/hh700485) before registering any of the background trigger types.

To ensure that your Universal Windows app continues to run properly after you release an update, you must call [**RemoveAccess**](https://msdn.microsoft.com/library/windows/apps/hh700471) and then call [**RequestAccessAsync**](https://msdn.microsoft.com/library/windows/apps/hh700485) when your app launches after being updated. For more information, see [Guidelines for background tasks](guidelines-for-background-tasks.md).

## Define the method signature and return type


This method takes in the task entry point, task name, a pre-constructed background task trigger, and (optionally) a [**SystemCondition**](https://msdn.microsoft.com/library/windows/apps/br224834) for the background task. This method returns a [**BackgroundTaskRegistration**](https://msdn.microsoft.com/library/windows/apps/br224786) object.

> [!div class="tabbedCodeSnippets"]
> ```cs
> public static BackgroundTaskRegistration RegisterBackgroundTask(
>                                                 string taskEntryPoint, 
>                                                 string name,
>                                                 IBackgroundTrigger trigger,
>                                                 IBackgroundCondition condition)
> {
>     
>     // We’ll add code to this function in subsequent steps.
> 
> }
> ```
> ```cpp
> BackgroundTaskRegistration^ MainPage::RegisterBackgroundTask(
>                                              Platform::String ^ taskEntryPoint,
>                                              Platform::String ^ taskName,
>                                              IBackgroundTrigger ^ trigger,
>                                              IBackgroundCondition ^ condition)
> {
>     
>     // We’ll add code to this function in subsequent steps.
> 
> }
> ```

## Check for existing registrations


Check whether the task is already registered. It's important to check this because if a task is registered multiple times, it will run more than once whenever it’s triggered; this can use excess CPU and may cause unexpected behavior.

You can check for existing registrations by querying the [**BackgroundTaskRegistration.AllTasks**](https://msdn.microsoft.com/library/windows/apps/br224787) property and iterating on the result. Check the name of each instance – if it matches the name of the task you’re registering, then break out of the loop and set a flag variable so that your code can choose a different path in the next step.

> **Note**  Use background task names that are unique to your app. Ensure each background task has a unique name.

The following code registers a background task using the [**SystemTrigger**](https://msdn.microsoft.com/library/windows/apps/br224838) we created in the last step:

> [!div class="tabbedCodeSnippets"]
> ```cs
> public static BackgroundTaskRegistration RegisterBackgroundTask(
>                                                 string taskEntryPoint, 
>                                                 string name,
>                                                 IBackgroundTrigger trigger,
>                                                 IBackgroundCondition condition)
> {
>     //
>     // Check for existing registrations of this background task.
>     //
> 
>     foreach (var cur in BackgroundTaskRegistration.AllTasks)
>     {
> 
>         if (cur.Value.Name == name)
>         {
>             // 
>             // The task is already registered.
>             // 
> 
>             return (BackgroundTaskRegistration)(cur.Value);
>         }
>     }
>     
>     // We’ll register the task in the next step.
> }
> ```
> ```cpp
> BackgroundTaskRegistration^ MainPage::RegisterBackgroundTask(
>                                              Platform::String ^ taskEntryPoint,
>                                              Platform::String ^ taskName,
>                                              IBackgroundTrigger ^ trigger,
>                                              IBackgroundCondition ^ condition)
> {
>     //
>     // Check for existing registrations of this background task.
>     //
>     
>     auto iter   = BackgroundTaskRegistration::AllTasks->First();
>     auto hascur = iter->HasCurrent;
>     
>     while (hascur)
>     {
>         auto cur = iter->Current->Value;
>         
>         if(cur->Name == name)
>         {
>             // 
>             // The task is registered.
>             // 
>             
>             return (BackgroundTaskRegistration ^)(cur);
>         }
>         
>         hascur = iter->MoveNext();
>     }
>     
>     // We’ll register the task in the next step.
> }
> ```

## Register the background task (or return the existing registration)


Check to see if the task was found in the list of existing background task registrations. If so, return that instance of the task.

Then, register the task using a new [**BackgroundTaskBuilder**](https://msdn.microsoft.com/library/windows/apps/br224768) object. This code should check whether the condition parameter is null, and if not, add the condition to the registration object. Return the [**BackgroundTaskRegistration**](https://msdn.microsoft.com/library/windows/apps/br224786) returned by the [**BackgroundTaskBuilder.Register**](https://msdn.microsoft.com/library/windows/apps/br224772) method.

> **Note**  Background task registration parameters are validated at the time of registration. An error is returned if any of the registration parameters are invalid. Ensure that your app gracefully handles scenarios where background task registration fails - if instead your app depends on having a valid registration object after attempting to register a task, it may crash.

The following example either returns the existing task, or adds code that registers the background task (including the optional system condition if present):

> [!div class="tabbedCodeSnippets"]
> ```cs
> public static BackgroundTaskRegistration RegisterBackgroundTask(
>                                                 string taskEntryPoint, 
>                                                 string name,
>                                                 IBackgroundTrigger trigger,
>                                                 IBackgroundCondition condition)
> {
>     //
>     // Check for existing registrations of this background task.
>     //
> 
>     foreach (var cur in BackgroundTaskRegistration.AllTasks)
>     {
> 
>         if (cur.Value.Name == taskName)
>         {
>             // 
>             // The task is already registered.
>             // 
> 
>             return (BackgroundTaskRegistration)(cur.Value);
>         }
>     }
>     
>     //
>     // Register the background task.
>     //
> 
>     var builder = new BackgroundTaskBuilder();
> 
>     builder.Name = name;
>     builder.TaskEntryPoint = taskEntryPoint;
>     builder.SetTrigger(trigger);
> 
>     if (condition != null)
>     {
> 
>         builder.AddCondition(condition);
>     }
> 
>     BackgroundTaskRegistration task = builder.Register();
> 
>     return task;
> }
> ```
> ```cpp
> BackgroundTaskRegistration^ MainPage::RegisterBackgroundTask(
>                                              Platform::String ^ taskEntryPoint,
>                                              Platform::String ^ taskName,
>                                              IBackgroundTrigger ^ trigger,
>                                              IBackgroundCondition ^ condition)
> {
> 
>     //
>     // Check for existing registrations of this background task.
>     //
>     
>     auto iter   = BackgroundTaskRegistration::AllTasks->First();
>     auto hascur = iter->HasCurrent;
>     
>     while (hascur)
>     {
>         auto cur = iter->Current->Value;
>         
>         if(cur->Name == name)
>         {
>             // 
>             // The task is registered.
>             // 
>             
>             return (BackgroundTaskRegistration ^)(cur);
>         }
>         
>         hascur = iter->MoveNext();
>     }
>     
>     //
>     // Register the background task.
>     //
> 
>     auto builder = ref new BackgroundTaskBuilder();
> 
>     builder->Name = name;
>     builder->TaskEntryPoint = taskEntryPoint;
>     builder->SetTrigger(trigger);
> 
>     if (condition != nullptr) {
>         
>         builder->AddCondition(condition);
>     }
> 
>     BackgroundTaskRegistration ^ task = builder->Register();
> 
>     return task;
> }
> ```

## Complete background task registration utility function


This example shows the completed background task registration function. This function can be used to register most background tasks, with the exception of networking background tasks.

> [!div class="tabbedCodeSnippets"]
> ```cs
> //
> // Register a background task with the specified taskEntryPoint, name, trigger,
> // and condition (optional).
> //
> // taskEntryPoint: Task entry point for the background task.
> // taskName: A name for the background task.
> // trigger: The trigger for the background task.
> // condition: Optional parameter. A conditional event that must be true for the task to fire.
> //
> public static BackgroundTaskRegistration RegisterBackgroundTask(string taskEntryPoint,
>                                                                 string taskName,
>                                                                 IBackgroundTrigger trigger,
>                                                                 IBackgroundCondition condition)
> {
>     //
>     // Check for existing registrations of this background task.
>     //
> 
>     foreach (var cur in BackgroundTaskRegistration.AllTasks)
>     {
> 
>         if (cur.Value.Name == taskName)
>         {
>             // 
>             // The task is already registered.
>             // 
> 
>             return (BackgroundTaskRegistration)(cur.Value);
>         }
>     }
>     
>     //
>     // Register the background task.
>     //
> 
>     var builder = new BackgroundTaskBuilder();
> 
>     builder.Name = taskName;
>     builder.TaskEntryPoint = taskEntryPoint;
>     builder.SetTrigger(trigger);
> 
>     if (condition != null)
>     {
> 
>         builder.AddCondition(condition);
>     }
> 
>     BackgroundTaskRegistration task = builder.Register();
> 
>     return task;
> }
> ```
> ```cpp
> //
> // Register a background task with the specified taskEntryPoint, name, trigger,
> // and condition (optional).
> //
> // taskEntryPoint: Task entry point for the background task.
> // taskName: A name for the background task.
> // trigger: The trigger for the background task.
> // condition: Optional parameter. A conditional event that must be true for the task to fire.
> //
> BackgroundTaskRegistration^ MainPage::RegisterBackgroundTask(Platform::String ^ taskEntryPoint,
>                                                              Platform::String ^ taskName,
>                                                              IBackgroundTrigger ^ trigger,
>                                                              IBackgroundCondition ^ condition)
> {
> 
>     //
>     // Check for existing registrations of this background task.
>     //
>     
>     auto iter   = BackgroundTaskRegistration::AllTasks->First();
>     auto hascur = iter->HasCurrent;
>     
>     while (hascur)
>     {
>         auto cur = iter->Current->Value;
>         
>         if(cur->Name == name)
>         {
>             // 
>             // The task is registered.
>             // 
>             
>             return (BackgroundTaskRegistration ^)(cur);
>         }
>         
>         hascur = iter->MoveNext();
>     }
> 
> 
>     //
>     // Register the background task.
>     //
> 
>     auto builder = ref new BackgroundTaskBuilder();
> 
>     builder->Name = name;
>     builder->TaskEntryPoint = taskEntryPoint;
>     builder->SetTrigger(trigger);
> 
>     if (condition != nullptr) {
>         
>         builder->AddCondition(condition);
>     }
> 
>     BackgroundTaskRegistration ^ task = builder->Register();
> 
>     return task;
> }
> ```

> **Note**  This article is for Windows 10 developers writing Universal Windows Platform (UWP) apps. If you’re developing for Windows 8.x or Windows Phone 8.x, see the [archived documentation](http://go.microsoft.com/fwlink/p/?linkid=619132).

 
## Related topics


****

* [Create and register a background task](create-and-register-a-background-task.md)
* [Declare background tasks in the application manifest](declare-background-tasks-in-the-application-manifest.md)
* [Handle a cancelled background task](handle-a-cancelled-background-task.md)
* [Monitor background task progress and completion](monitor-background-task-progress-and-completion.md)
* [Respond to system events with background tasks](respond-to-system-events-with-background-tasks.md)
* [Set conditions for running a background task](set-conditions-for-running-a-background-task.md)
* [Update a live tile from a background task](update-a-live-tile-from-a-background-task.md)
* [Use a maintenance trigger](use-a-maintenance-trigger.md)
* [Run a background task on a timer](run-a-background-task-on-a-timer-.md)
* [Guidelines for background tasks](guidelines-for-background-tasks.md)

****

* [Debug a background task](debug-a-background-task.md)
* [How to trigger suspend, resume, and background events in Windows Store apps (when debugging)](http://go.microsoft.com/fwlink/p/?linkid=254345)

 

 



