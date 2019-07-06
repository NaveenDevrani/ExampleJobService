# ExampleJobService

# JobService

JobService is an android service component with callback methods which the JobSchedule calls when a job needs to be run. That means your background job code needs to be added to callback methods of JobService. To create JobService, you need to extend JobService class and implement its methods **onStartJob** and **onStopJob**.

These callback methods are passed JobParameters as an argument by the JobScheduler. 
JobParameters contains parameters that are set when job is scheduled. 
These call back methods return boolean value,
> **false indicates that the job is complete**    
                     **and **   
> **true tells that job is still running in the background**,      
 in this case you need to call jobFinished method to inform job scheduler about the finished state after the job is complete.



Since job service is an android component, it runs on main thread. That’s why the tasks which are in job service callback methods needs to be run on background thread.

 public class DownloadJobService extends JobService{

    @Override
    public boolean onStartJob(JobParameters jobParameters) {
	
	downloadFile();
        return false;
    }
    @Override
    public boolean onStopJob(JobParameters jobParameters) {
        return false;
    }
} 


Since it is a service, you need to define it in the manifest xml file and it needs to be protected with android.permission.BIND_JOB_SERVICE permission.

<service android:name=".DbUpdateJobService"
            android:permission="android.permission.BIND_JOB_SERVICE" ></service>
            
            
            
            
            Scheduling Job
To schedule a job, first you need to get JobScheduler instance by calling getSystemService on the context object passing JOB_SCHEDULER_SERVICE argument to it.

JobScheduler jobScheduler = (JobScheduler)getApplicationContext()
.getSystemService(JOB_SCHEDULER_SERVICE);
Then you need to instantiate ComponentName object by passing context and job service class that you created.

ComponentName componentName = new ComponentName(this,
        DownloadJobService.class);
Then you can create JobInfo object using JobInfo.Builder and setting various configuration parameters. JobInfo.Builder has various setter methods which allow you to define your Job. See the following sections for more details.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
.setPeriodic(50000).setRequiresBatteryNotLow(true).build();
Then finally, call schedule() on job scheduler object passing job info object to it.

 jobScheduler.schedule(jobInfo);
Creating JobInfo
As shown above JobInfo is created using JobInfo.Builder class by using its setter methods, following are the details.

To create a job that runs every time after elapsing of a specified time, you can use setPeriodic method passing time in milliseconds.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
.setPeriodic(50000).build();
To create a job that needs to be rerun if it fails, you need to use setBackOffCriteria() method passing time interval for the first time retry and retry policy which is used to calculate time interval for retries after first retry.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
      .setBackoffCriteria(6000, JobInfo.BACKOFF_POLICY_LINEAR).build();
You can make a job delayed by specified amount of time by setting minimum latency and you can also specify maximum delay by calling setOverrideDeadline. These two setting are applicable for only non-periodic jobs.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
        .setMinimumLatency(500).setOverrideDeadline(300).build();
To create a job that stays as scheduled even after device reboots, you need call setPersisted() method on JobInfo.Builder passing true as value to it. This setting requires RECEIVE_BOOT_COMPLETED permission.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
        .setPersisted(true).build()
If your job needs network connection and you want to run the job when network of specified kind is used, then you need to specify required network type by calling setRequiredNetworkType() method passing network type.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
        .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED).build();

 
You can configure your job to run when the device is charging and idle by calling setRequiresCharging and setRequiresDeviceIdle methods respectively or you can ask the scheduler to not run the job when battery is low by calling setRequiresBatteryNotLow() method and passing true.

Similarly, you can schedule a job to run only when storage is not low by calling setRequiresStorageNotLow() method and setting it to true.

 JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
        .setRequiresCharging(true)
        .setRequiresDeviceIdle(true)
        .setRequiresBatteryNotLow(true).build();
To pass data to JobService, you need to set extras by creating PersistableBundle object. JobScheduler passes PersistableBundle object to JobService call back methods.

 	PersistableBundle pb = new PersistableBundle();
	    pb.putBoolean("cashback" , false);
	    pb.putDouble("min", 20.3);
	    pb.putString("exclude", "deals");

	JobInfo jobInfoObj = new JobInfo.Builder(1, componentName)
			.setExtras(pb).build();
