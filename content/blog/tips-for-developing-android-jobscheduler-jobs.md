+++
date = "2018-10-17T08:31:27+01:00"
draft = true
title = "Tips for developing Android JobScheduler Jobs"

+++
If your app has to sync some data in the background, you'll definitely want to use the Android JobScheduler API to schedule a background job.

There are a few helpful libraries that make this task easier, but you can get away with using the JobScheduler API itself.

I work in Xamarin, so my samples are in C#.

## Scheduling a Job

The JobScheduler has a very simple API and you'll probably won't need more than one or two methods: 

```C#
var jobScheduler = (JobScheduler) context.GetSystemService(Context.JobSchedulerService);

var jobSchedulingResult = jobScheduler.Schedule(
    new JobInfo.Builder(1234, new ComponentName(context, Class.FromType(SyncJobService))) // 1234: is the unqiue ID of this job in your app - SyncJobService is the type of your JobService
        .SetPeriodic(1000*60*15) // in ms
        .SetRequiredNetworkType(NetworkType.Any)
        .Build());

if (jobSchedulingResult == JobScheduler.ResultSuccess)
{
    // yay
}
else
{
    // :(
}
```

This method schedules a new job or updates an existing one. There's one catch: it will cancel any running job with the same ID. Also: Android decides when to run your job. It will try to respect the periodic interval, but your job won't run more often than every 15 minutes.

So before scheduling our job, we might have to check if the job is already scheduled with the correct parameters so that we don't reschedule it unnecessarily.

```C#
var pendingJob = jobScheduler.GetPendingJob(1234);
if (pendingJob == null)
{
    // maybe verify the job settings here
}
```

A Job can have a multitide of parameters (_constraints_) that the JobScheduler uses to determine when the Job should run:

* Device is idle
* Low battery
* Periodic
* Network type

There are methods on the `JobInfo.Builder` class for each of these parameters.

## Implementing the JobService

`JobService` is the base class for any job.

```C#
[Service(Permission = PermissionBind)]
public class SyncJobService : JobService
{
    public override bool OnStartJob(JobParameters @params)
    {
        StartSync();
        return false;
    }

    public override bool OnStopJob(JobParameters @params)
    {
        return false;
    }
}
```

When your job has to start, the method `OnStartJob` is called.  The `params` have some information like which network should be used for your job or which constraints triggered the job. Note that this method runs on the main (UI) thread, which is often not what you want. So you should use something like `Task.Run(() => StartSync(), TaskCreationOptions.LongRunning)` to make sure it runs on a background thread.

The return value of `OnStartJob` indicates if the job will still be running after returning from this method. You have to explicitly call `JobFinished(@params, false)` to indicate that the job is finished. The boolean in this method indicates if the job has to be rescheduled (because if failed) and same goes for the boolean in `OnStopJob`. The method `OnStopJob` is called when your job has to be cancelled.

## Combining with BroadcastReceivers

The app I'm building requires the job to be always scheduled, so I used a BroadcastReceiver to schedule the job when the device boots up.

```C#
[assembly: UsesPermission(Manifest.Permission.ReceiveBootCompleted)]

[BroadcastReceiver(Enabled = true, Exported = true, DirectBootAware = true)]
[IntentFilter(new[] {Intent.ActionLockedBootCompleted})]
public class BootedBroadcastReceiver: MvxBroadcastReceiver
{
    public override void OnReceive(Context context, Intent intent)
    {
        this.RegisterSetupType<Setup>();
        base.OnReceive(context, intent);
        
        ScheduleJobs(context);
    }
}
```

This MvvmCross base class makes sure that the `Setup` has completed before scheduling your jobs. Don't forget the required permission to receive `BootCompleted` intents.

I also schedule the job when the app opens so that users don't have to reboot their device after installing the app.

## Processes

Most of my apps have an IoC container or some services that have to be initialized/restored before the app can do anything. If you use MvvmCross, you can use `MvxAndroidSetupSingleton.EnsureSingletonAvailable(this).EnsureInitialized()` in `OnStartJob` to make sure everything is initialized. Note that jobs - by default - run in the same process that scheduled the jobs in the first. If you reschedule your jobs, then that process becomes the host.

So with all of the above, this is what happens:

1. Device boots up - BroadcastReceivers schedules the jobs (process 1)
1. The JobScheduler executes the job in process 1
1. The user opens the app, but Android detects that there's also a process for this app (process 1)
1. As the setup has already ran in this process, the app opens a lot faster

So that makes for a very helpful side effect: app start up is a lot faster since the setup has already completed to make sure that the jobs can run.

There are attributes to mark that your job has to run in an isolated process, but that usually involves a lot more work to make sure that your data (database, local storage) can work with this. Systems like `MvxMessenger` also don't work between multiple processes.

## Some helpful commands

You can try and test this on a emulator, but I recommend to use Android 7.1 or newer for this. You can't force any jobs to run immediately while testing on versions before 7.1.

### Monitor when adb becomes available

So that you can connect to logcat as soon as ADB becomes available during the boot process.

```
while :; do; clear; adb devices; sleep 2; done
```

### Info about scheduled jobs

Use grep to search for the history of executed jobs or for the schedule of your jobs.

```
adb shell dumpsys jobscheduler
adb shell dumpsys jobscheduler | grep -C 5 your.package.id
adb shell dumpsys jobscheduler | grep -B 10 "Pending queue"
```

### Run a scheduled job instantly

End with your package ID and job ID.

```
adb shell cmd jobscheduler run -f your.package.id 1234
```