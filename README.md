[![Build Status](https://travis-ci.org/vineetchoudhary/iOS-Universal-Links.svg?branch=master)](https://travis-ci.org/vineetchoudhary/iOS-Universal-Links)

# iOS Universal Links
Universal Links, iOS 9 users can tap a link to your website and get seamlessly redirected to your installed app without going through Safari. If your app isn’t installed, tapping a link to your website opens your website in Safari.

# Support Universal Links
When you support universal links, iOS 9 users can tap a link to your website and get seamlessly redirected to your installed app without going through Safari. If your app isn’t installed, tapping a link to your website opens your website in Safari.

I’ll show how to setup your own server, and handle the corresponding links in your app

## Server Setup
You need to having a server running online. To securely associate your iOS app with a server, Apple requires that you make available a configuration file, called apple-app-site-association. This is a JSON file which describes the domain and supported routes.

The apple-app-site-association file needs to be accessible via HTTPS, without any redirects, at https://{domain}/apple-app-site-association. 

The file looks like this:

    {
    "applinks": {
        "apps": [ ],
        "details": [
            {
                "appID": "{app_prefix}.{app_identifier}",
                "paths": [ "/path/to/content", "/path/to/other/*", "NOT /path/to/exclude" ]
            },
            {
                "appID": "TeamID.BundleID2",
                "paths": [ "*" ]
            }
        ]
    }
    }
    

__NOTE__ - Don’t append .json to the apple-app-site-association filename.


The keys are as follows:   
`apps`: Should have an empty array as its value, and it must be present. This is how Apple wants it.  
`details`: Is an array of dictionaries, one for each iOS app supported by the website. Each dictionary contains information about the app, the team and bundle IDs.

There are 3 ways to define paths:   
`Static`: The entire supported path is hardcoded to identify a specific link, e.g. /static/terms  
`Wildcards`: A * can be used to match dynamic paths, e.g. /books/* can matches the path to any author’s page. ? inside specific path components, e.g. books/1? can be used to match any books whose ID starts with 1.  
`Exclusions`: Prepending a path with NOT excludes that path from being matched.

The order in which the paths are mentioned in the array is important. Earlier indices have higher priority. Once a path matches, the evaluation stops, and other paths ignored. Each path is case-sensitive.

### Supporting Multiple Domains
Each domain supported in the app needs to make available its own apple-app-site-association file. If the content served by each domain is different, then the contents of the file will also change to support the respective paths. Otherwise, the same file can be used, but it needs to be accessible at every supported domain.

### Signing the App-Site-Association File
If your app targets iOS 9 and your server uses HTTPS to serve content, you don’t need to sign the file. If not (e.g. when supporting Handoff on iOS 8), it has to be signed using a SSL certificate from a recognized certificate authority.

__Note__: This is not the certificate provided by Apple to submit your app to the App Store. It should be provided by a third-party, and it’s recommended to use the same certificate you use for your HTTPS server (although it’s not required).

To sign the file, first create and save a simple .txt version of it. Next, in the terminal, run the following command:

    cat <unsigned_file>.txt | openssl smime -sign -inkey example.com.key -signer example.com.pem -certfile intermediate.pem -noattr -nodetach -outform DER > apple-app-site-association
    
This will output the signed file in the current directory. The example.com.key, example.com.pem, and intermediate.pem are the files that would made available to you by your Certifying Authority.

__Note__: If the file is unsigned, it should have a Content-Type of application/json. Otherwise, it should be application/pkcs7-mime.


The website code can be found gh-pages branch on https://github.com/vineetchoudhary/iOS-Universal-Links/tree/gh-pages

## App Setup
Next is the app, that mirrors the server. It will target iOS 9 and will be using Xcode 7.2 with Objective-C.

The app code can be found master branch on https://github.com/vineetchoudhary/iOS-Universal-Links/

### Enabling Universal Links
The setup on the app side requires two things:
  - Configuring the app’s entitlement, and enabling universal links.
  - Handling Incoming Links in your AppDelegate.
  
#### Configuring the app’s entitlement, and enabling universal links.
The first step in configuring your app’s entitlements is to enable it for your App ID. Do this in the Apple Developer Member Center. Click on Certificates, Identifiers & Profiles and then Identifiers. Select your App ID (create it first if required), click Edit and enable the Associated Domains entitlement.
![](/MC-Domain.png)

Next, get the App ID prefix and suffix by clicking on the respective App ID.

The App ID prefix and suffix should match the one in the apple-app-site-association file.

Next in Xcode, select your App’s target, click Capabilities and toggle Associated Domains to On. Add an entry for each domain that your app supports, prefixed with applinks:.

For example: applinks:vineetchoudhary.github.io

Which looks like this for the sample app:
![](/App-Domain.png)

__Note__: Ensure you have selected the same team and entered the same Bundle ID as the registered App ID on the Member Center. Also ensure that the entitlements file is included by Xcode by selecting the file and in the File Inspector, ensure that your target is checked.  
![](/target.png)



#### Handling Incoming Links in your AppDelegate.
`[UIApplicationDelegate application: continueUserActivity: restorationHandler:]` method in AppDelegate.m handles incoming links.  You parse this URL to determine the right action in the app.

For example, in the sample app:

    -(BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler{
        if ([userActivity.activityType isEqualToString: NSUserActivityTypeBrowsingWeb]) {
            NSURL *url = userActivity.webpageURL;
            UIStoryboard *storyBoard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
            UINavigationController *navigationController = (UINavigationController *)_window.rootViewController;
            if ([url.pathComponents containsObject:@"home"]) {
                [navigationController pushViewController:[storyBoard instantiateViewControllerWithIdentifier:@"HomeScreenId"] animated:YES];
            }else if ([url.pathComponents containsObject:@"about"]){
                [navigationController pushViewController:[storyBoard instantiateViewControllerWithIdentifier:@"AboutScreenId"] animated:YES];
            }
        }
        return YES;
    }  
    
### DONE

![](/screen.gif)

### References 
https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html
http://www.developerinsider.in/enable-universal-links-in-ios-app-and-setup-server-for-it/
