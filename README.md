## Tutorial

1. Add library to project.
	File -> Import… -> General -> Existing project into workspace
	Select root directory of InAppBlast library by pressing Browse… button, then click on Finish button.
	Now select in Package explorer Your project where you going to add InAppBlast library. On Eclipse menu bar Project -> Properties -> Android 
	In Library section click on Add… button select InAppBlast library and press Ok button
	Now InAppBlast library added to the project.

2.  Library usage
	Library can be used in application with minimal API level 8. 
	In order to use library the following steps must be accomplished:

3. Create subclass of Application class. Implement in this class OnBlastEventListener interface. Override onCreate method and initialize AppBlast in this method.

		public class MyApplication extends Application implements OnBlastActionListener {
			    
			@Override
			public void onCreate() {
				super.onCreate();
				// Initialize
				AppBlast.initSharedInstance(<project_key>, this);
				// Set user id
				AppBlast.getSharedInstance().setUserIdIfNotSet(<user_id>);
				// Set log level
				AppBlast.getSharedInstance().setLogLevel(AppBlast.LL_ALL);
			}
			    
			@Override
			public void onBlastAction(int action, Object... params) {
				switch (action) {
					case OnBlastActionListener.ACTION_POSITIVE:
						String url = (String) params[0];
						Log.i("In application action result", "CTA URL = " + url);
						break;

					case OnBlastActionListener.ACTION_NEGATIVE:
						Log.i("In application action result", "Closed or back pressed");
						break;
				}	 
			}
		}

4. In AndroidManifest file:
	- Add ActBlast activity 

		...
		<activity android:name="com.inappblast.ActBlast"></activity>
		...

	- Add following permissions 

		...
		<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
		<uses-permission android:name="android.permission.INTERNET"/>
		...

	- Use subclassed Application class. (In application tag set name attribute) 

		...
		<application android:name="MyApplication"
		...

5. During runtime all activities that goes to foreground should be registered with AppBlast instance. There is two ways to do that.
	- In each activity override onResume() and onPause() methods. And call AppBlast.getSharedInstance().setBlastPoint();

		@Override
		protected void onResume() {
			super.onResume();
			AppBlast.getSharedInstance().setBlastPoint(this);
		}

		@Override
		protected void onPause() {
			super.onPause();
			AppBlast.getSharedInstance().setBlastPoint(null);
		}

	- Starting from API level 14 we can use Application.ActivityLifecycleCallbacks interface that can help register foreground applications with AppBlast instance. Implement ActivityLifecycleCallbacks interface and register that implementation with Application.

		public class MyApplication extends Application implements OnBlastActionListener, ActivityLifecycleCallbacks {

			@Override
			public void onCreate() {
				super.onCreate();
				// AppBlastInitialization
				AppBlast.initSharedInstance(<project_key>, this);
				AppBlast.getSharedInstance().setUserIdIfNotSet(<user_id>);
				AppBlast.getSharedInstance().setLogLevel(AppBlast.LL_ALL);

				// Register ActivityLifecycleCallbacks
				registerActivityLifecycleCallbacks(this);
			}

			/*
			 * Blast action listener
			 */
			@Override
			public void onBlastAction(int action, Object... params) {
				switch (action) {
				case OnBlastActionListener.ACTION_POSITIVE:
					// On positive action
					break;

				case OnBlastActionListener.ACTION_NEGATIVE:
					// On negative action
					break;
				}
			}

			/*
			 * Activity lifecycle callbacks
			 */

			@Override
			public void onActivityCreated(Activity activity, Bundle savedInstanceState) {}

			@Override
			public void onActivityStarted(Activity activity) {}

			@Override
			public void onActivityResumed(Activity activity) {
				AppBlast.getSharedInstance().setBlastPoint(activity);
			}

			@Override
			public void onActivityPaused(Activity activity) {
				AppBlast.getSharedInstance().setBlastPoint(null);
			}

			@Override
			public void onActivityStopped(Activity activity) {}

			@Override
			public void onActivitySaveInstanceState(Activity activity, Bundle outState) {}

			@Override
			public void onActivityDestroyed(Activity activity) {}
		}

	- If you don't want to manage Blast actions manually, just remove OnBlastActionListener-interface implementation from you class.
