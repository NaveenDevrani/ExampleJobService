# ExampleJobService

#JobService

JobService is an android service component with callback methods which the JobSchedule calls when a job needs to be run. That means your background job code needs to be added to callback methods of JobService. To create JobService, you need to extend JobService class and implement its methods #onStartJob and #onStopJob.
