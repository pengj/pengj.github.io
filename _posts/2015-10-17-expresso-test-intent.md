---
layout: post
title: Android Espresso Test for Intent
categories: [android, test]
tags: [android, test, espresso, intent]
fullview: true
comments: true
---

## Intent Test

> An intent is an abstract description of an operation to be performed.

You all know `Intent`, and you also know it works with `startActivity` or `startActivityForResult` to start another activity or app to get result. How to test these behaviours in the UI test is a little tricky, because we cannot manipulate the user interface of an external activity nor control the ActivityResult returned to the activity under test. Luckily, [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/index.html) has a nice package [Intents](https://google.github.io/android-testing-support-library/docs/espresso/intents/index.html), which handles this problem. There are three situations you need to think about for the [Intent](http://developer.android.com/training/basics/intents/index.html) test and I will describe an example for each situation. 

1. Sending the user to another App
2. Getting a Result from an `Activity`
3. Allowing Other Apps to start your `Activity`


### Setup dependence

The `Intents`package is not in the default `Espresso` package, you need to add it with other `Espresso` packages: the core, the rules, and the runner: 

{% highlight yaml %}
androidTestCompile 'com.android.support.test:runner:0.4.1'
androidTestCompile 'com.android.support.test:rules:0.4.1'
androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
androidTestCompile 'com.android.support.test.espresso:espresso-intents:2.2.1'
{% endhighlight %}

### External Intent

> External intent is used to send user to another app to get result: such as open camera to take a picture or open contact to select an email. 

Here is an example: user select an image from a gallery app, then your app displays the selected image in an `ImageView`. The Intent to trigger the event will be something like this:

{% highlight java %}
Intent intent = new Intent(Intent.ACTION_PICK,
               android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
startActivityForResult(intent, MainActivity.RESULT_LOAD_IMAGE);
{% endhighlight %}

What you want to test in your `onActivityResult` is you receive a validated image `Uri` (the selected image is parsed as `Uri`), you can display it in the `ImageView`. You can't control which app user use to select the image, as long as it will return a validated `Uri` in the result. 

First, let's use `Instrumentation` to create a `ActivityResult`. Here I use launch icon as test image and parse it as an `Uri`. 

{% highlight java %}
 Resources resources = InstrumentationRegistry.getTargetContext().getResources();
 Uri imageUri = Uri.parse(ContentResolver.SCHEME_ANDROID_RESOURCE + "://" + 
 		resources.getResourcePackageName(R.mipmap.ic_launcher) + '/' + 
                resources.getResourceTypeName(R.mipmap.ic_launcher) + '/' + 
                resources.getResourceEntryName(R.mipmap.ic_launcher));
        
Intent resultData = new Intent();
resultData.setData(imageUri);
Instrumentation.ActivityResult result = new Instrumentation.ActivityResult(
                Activity.RESULT_OK, resultData);
{% endhighlight %}

There are two methods you need to understand when using `Intents`: `intending` and `intended`. If you use `Mockito`, they are very similar. 

* `intending` is like `when` and `respondWith` is like `thenReturn`
* `intended` is like `verify`, assert that a given intent has been seen

Before you can use these two methods, you need to initialize the `Intents`, which is not clear when I first use the `Intents`, otherwise you will get a `NullPointerException` Exception.

In the `Mockito`, you will use method name to match the action. In the `Intents`, you need to use `Matcher<Intent>` to match the `Intent`. In the existing intent matcher, it has many methods responding to the `Setter` you can use for the `Intent`:

* Intent.setData <--> hasData
* Intent.setAction <--> hasAction
* Intent.setFlag <--> hasFlag
* Intent.setComponent <--> hasComponent

Here is the code to match the image select `Intent`:

{% highlight java %}
Matcher<Intent> expectedIntent = allOf(hasAction(Intent.ACTION_PICK),
       hasData(android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI));
Intents.init();
intending(expectedIntent).respondWith(result);

 //Click the select button
onView(withId(R.id.fab_image)).perform(click());
intended(expectedIntent);
Intents.release();

//Check the image is displayed
onView(withId(R.id.imageView)).check(matches(hasDrawable()));
{% endhighlight %}
 
If you want to implement your own intent matcher, you can check this [tutorial](http://code.google.com/p/hamcrest/wiki/Tutorial)

### Internal Intent

> Internal Intent is used to getting a result from an Activity in the same app: such as login, select an item. 

This situation is easy to test because you control the interaction, but you need to consider all the situation such as user cancel the action (press back button). 

Here is an example to test what happen if user cancel the login action. 

{% highlight java %}
Instrumentation.ActivityResult activityResult = new Instrumentation.ActivityResult(
                Activity.RESULT_CANCELED, new Intent());

Intents.init();
intending(expectedIntent).respondWith(activityResult);
onView(withId(R.id.button_login)).perform(click());
intended(expectedIntent);
Intents.release();

onView(withId(R.id.button_login)).check(matches(withText(R.string.cancel_login)));
{% endhighlight %}


### Share Intent

> Allowing other Apps to Start your app and perform an action that might be useful to another app

There are two parts you want to test:

1. You app responses to the correct Intent
2. You app does the correct action when receive the validated incoming data  

>Caution: Take extra care to check the incoming data, you never know what other application may send you. 

{% highlight java %}
// Setup the trigger Intent
Intent intent = new Intent();
intent.setAction(Intent.ACTION_SEND);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
intent.setType(TextHashActivity.SUPPORT_TYPE);
intent.putExtra(Intent.EXTRA_TEXT, TEST_INPUT);

// Need to set the Intent explicit here, because it may have more than one app to handle this action and we don't want to have the App chooser here

String packageName = InstrumentationRegistry.getTargetContext().getPackageName();
ComponentName componentName = new ComponentName(packageName,
                TextHashActivity.class.getName());
intent.setComponent(componentName);

Intents.init();
// start activity from test app
InstrumentationRegistry.getContext().startActivity(intent);

// Check the intent is handled by the app
Matcher<Intent> expectedIntent = hasComponent(componentName);
intended(expectedIntent);
Intents.release();

// Check the action is correctly performed 
onView(withId(R.id.textview_sha1))
                .check(matches(withText(TEST_SHA1)));
{% endhighlight %} 

### Sample Source code:

You can find the sample source code in [Github](https://github.com/pengj/Intent-Test). The sample app has following actions: 

1. Pick up an image from gallery app and display in an `ImageView`
2. Login with user name and password (using android studio Login Activity)
3. User input some text and display SHA1 value of the text
4. Another app send some text and display SHA1 value of the text

### Read More: 

If you want to know more about Espresso Intent test, these post and sample code help me out a lot when writing this post:

1. [Testing-intents-with-espresso-intents](http://michaelevans.org/blog/2015/09/15/testing-intents-with-espresso-intents/)
2. [Stub-your-android-intents](http://collectiveidea.com/blog/archives/2015/08/11/stub-your-android-intents/)
3. [Espresso sample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso)
4. [Espresso Intent sample](https://github.com/googlesamples/android-testing/tree/master/ui/espresso/IntentsBasicSample)