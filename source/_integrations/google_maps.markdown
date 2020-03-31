---
title: Google Maps
description: Instructions how to use Google Maps Location Sharing to track devices in Home Assistant.
ha_release: 0.67
ha_category:
  - Presence Detection
ha_iot_class: Cloud Polling
ha_domain: google_maps
---

The `google_maps` platform allows you to detect presence using the unofficial API of [Google Maps Location Sharing](https://myaccount.google.com/locationsharing).

## Setup

You first need to create an additional Google account and share your location with that account. This integration will use that account to fetch the location of your device(s). 

1. You have to setup sharing through the Google Maps app on your mobile phone. You can find more information [here](https://support.google.com/accounts?p=location_sharing).
2. You need to use the cookies from that account after you have properly authenticated which you can retrieve with:
    * Firefox:
      * Use the add-on "[Export cookies](https://addons.mozilla.org/en-US/firefox/addon/export-cookies-txt/)", but make sure that "Prefix HttpOnly cookies" is unchecked
      * If you are using Firefox Containers, you may need to use the add-on "[Cookie Quick Manager](https://addons.mozilla.org/firefox/addon/cookie-quick-manager/)" instead, however, this is not recommended, as it will only export or copy the cookies to the clipboard in JSON format (until [a feature request is implemented to resolve it](https://github.com/ysard/cookie-quick-manager/issues/94)). You will need to rewrite these values using these PCRE-regex statements:
      ```
      s/\[?\{\s+"Host raw": "(http[s]*):\/\/([^\/]+)\/",\s+"Name raw": "([^"]+)",\s+"Path raw": "([^"]+)",\s+"Content raw": "([^"]+)",\s+"Expires": "[^"]+",\s+"Expires raw": "([^"]+)",\s+"Send for": "[^"]+",\s+"Send for raw": "[^"]*",\s+"HTTP only raw": "[^"]*",\s+"SameSite raw": "[^"]*",\s+"This domain only": "([^"]+)",\s+"This domain only raw": "[^"]*",\s+"Store raw": "[^"]*",\s+"First Party Domain": "[^"]*"\s+\}\,*\]?/$2\t$7\t$4\t$1\t$6\t$3\t$5/gm
      s/(https|Valid for host only)/TRUE/g
      s/(http|Valid for subdomains)/FALSE/g
      ```
    * Chrome/Chromium based browsers
      * Use the extension [cookies.txt](https://chrome.google.com/webstore/detail/cookiestxt/njabckikapfpffapmjgojcnbfjonfjfg?hl=en-US) 
3. Save the cookie file to your Home Assistant configuration directory with the following name: `.google_maps_location_sharing.cookies.` followed by the slugified username of the NEW Google account. Make sure to use the `.com` TLD (e.g., maps.google.com), otherwise the cookie won't be able to provide a valid session.
   - For example: If your email address was `location.tracker@gmail.com`, the filename would be: `.google_maps_location_sharing.cookies.location_tracker_gmail_com`.

## Configuration

To integrate Google Maps Location Sharing in Home Assistant, add the following section to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
device_tracker:
  - platform: google_maps
    username: YOUR_USERNAME
```

Once enabled and you have rebooted devices discovered through this integration will be listed in the `known_devices.yaml` file within your configuration directory.

They will be created with indentifiers like `google_maps_<numeric_id>`. To be able to properly track entities you must set the `track` attribute to `true`. 

{% configuration %}
username:
  description: The email address for the Google account that has access to your shared location.
  required: true
  type: string
max_gps_accuracy:
   description: Sometimes Google Maps can report GPS locations with a very low accuracy (few kilometers). That can trigger false zoning. Using this parameter, you can filter these false GPS reports. The number has to be in meters. For example, if you put 200 only GPS reports with an accuracy under 200 will be taken into account - Defaults to 100km if not specified.
   required: false
   type: float
scan_interval:
  description: The frequency (in seconds) to check for location updates.
  required: false
  default: 60
  type: integer
{% endconfiguration %}

<div class='note'>
As of release 0.97 Google passwords are no longer required in your configuration. Users coming from earlier releases should only remove the password entry from their configuration file (username is still required) and restart Home Assistant. The cookie file previously generated should still be valid and will allow the tracker to continue functioning normally until the cookie is invalidated.
</div>
