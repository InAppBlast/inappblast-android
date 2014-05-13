## Tutorial

1. Add library to project.
	File -> Import... -> General -> Existing project into workspace.
	Select root directory of InAppBlast library by pressing Browse... button, then click on Finish button.
	Now select in Package explorer Your project where you going to add InAppBlast library. On Eclipse menu bar Project -> Properties -> Android.
	In Library section click on Add... button select InAppBlast library and press Ok button.
	Now InAppBlast library added to the project.

2. Library can be used in application with minimal API level 8.

3. Get your <project_key> string (just drop us an email to hi@pushpanel.io to get your key).

4. Create subclass of Application class. Override onCreate method and initialize AppBlast in this method.

		public class MyApplication extends Application {
			@Override
			public void onCreate() {
				super.onCreate();
				...
				AppBlast.initSharedInstance(<project_key>, this);
			}
		}

5. To identify your user within InAppBlast (either you use user authentication method or no authentication at all), put the following line where appropriate (as a way of "identifying" your user inside InAppBlast object).

		AppBlast.getSharedInstance().setUserIdIfNotSet(<user_id>);

6. In AndroidManifest file:
	Add ActBlast activity

		<activity android:name="com.inappblast.ActBlast"></activity>

	Add following permissions

		<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
		<uses-permission android:name="android.permission.INTERNET"/>

	Use subclassed Application class. (In application tag set name attribute)

		<application
			android:name="MyApplication"

7. During runtime all activities that goes to foreground should be registered with AppBlast instance. There is two ways to do that.
	In each activity override onResume() and onPause() methods. And call AppBlast.getSharedInstance().setBlastPoint();

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

	Starting from API level 14 we can use Application.ActivityLifecycleCallbacks interface that can help register foreground applications with AppBlast instance. Implement ActivityLifecycleCallbacks interface and register that implementation with Application.

		public class MyApplication extends Application implements ActivityLifecycleCallbacks {
			@Override
			public void onCreate() {
				super.onCreate();
				...
				AppBlast.initSharedInstance(<project_key>, this);
				AppBlast.getSharedInstance().setUserIdIfNotSet(<user_id>);
				AppBlast.getSharedInstance().setLogLevel(AppBlast.LL_ALL);
				registerActivityLifecycleCallbacks(this);
			}

			...

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

8. If you want to manage Blast actions manually, just implement OnBlastActionListener interface from you class.

		public class MyApplication extends Application implements OnBlastActionListener {
			...
			@Override
			public void onBlastAction(int action, Object... params) {
				switch (action) {
					case OnBlastActionListener.ACTION_POSITIVE:
						String url = (String) params[0];
						Log.i("In application action result", "CTA URL = " + url); // for example
						break;

					case OnBlastActionListener.ACTION_NEGATIVE:
						Log.i("In application action result", "Closed or back pressed"); // for example
						break;
				}
			}
			...
		}

9. You can change InAppBlast log level by calling setLogLevel method.

		AppBlast.getSharedInstance().setLogLevel(AppBlast.LL_ALL); // or AppBlast.LL_NO or AppBlast.LL_ERROR
