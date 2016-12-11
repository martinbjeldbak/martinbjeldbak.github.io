---
title: "fonecal: A calendar generator for the 2014 Formula 1 season"
date: 2014-03-18 22:46:36 +0100
comments: true
<!-- categories: ruby -->
---

I love Formula 1. This season is especially exciting, hence there's not a qualifying session or race that I want to miss. Unfortunately my search for any good, sensible calendars that simply include practice, qualifying, and race events in my timezone regardless of the event's timezones has been fruitless. So I wrote my own calendar parser for this 2014 Formula One season.

You can find the iCalendar file for the entire racing season here: (OUTDATED, contact me if interested). If you wish, you can merely download this file in a browser and open it in a text editor or calendar program to check out its contents. The code generating this file is open source and can be found on GitHub via a link below so if you don't trust me, you can inspect it and make sure I'm not doing anything shady.

### Subscribing to f1cal in Google Calendar
If you use Google Calendar I suggest you subscribe to it by adding the calendar via its URL by doing the following:

1. Click the arrow to the right of "other calendars" in the left pane
2. Click on Add by URL
3. Paste in the URL (OUTDATED, contact me if interested)
4. Profit

For official instructions, see [this](https://support.google.com/calendar/answer/37100?hl=en) link from Google. This creates a new, separate calendar with practice, qualifying, and race sessions for each grand prix in your calendar and in your time zone. By subscribing to the calendar, your calendar will automatically update if any of the events change, e.g. a practice session is delayed one hour because of heavy rain or whatever.

### Implementation

Appropriately named f1cal ([GitHub link](https://github.com/{{ site.author.github }}/fonecal)), this still unreleased gem is written in Ruby and crawls all the grand prix links in (link here no longer working), and for each of these grand prix this season, it extracts metadata about each event belonging to the grand prix.

It uses the amazing [geocoder](https://github.com/alexreisner/geocoder) gem to find the geological locations for each event, as this is impossible to tell merely from the event's metadata on formula1.com. This information is used to determine the timezone of the event so that, when the iCal file is built, it can properly be converted to UTC time allowed by the iCalendar standard (see [here](http://www.kanzaki.com/docs/ical/dateTime.html)). This has the benefit of letting the user's calendar locational information decide what time in the user's timezone the event is.

To generate the iCalendar file, simply import the fonecal gem into your runtime and write
{% highlight ruby %}
Fonecal.create_ical
{% endhighlight %}
to initiate the crawler and output a .ics (.ical) file in the current directory with all the event information.

It is still in very early phases of development. Having written it in a weekend with only a few tests, I'm sure there are quite a few bugs and that the interface could be cleaner. If you want to help, I'd really appreciate it!

### Future work
- Properly name the calendar event race title, e.g. 2014 Formula 1 Rolex Australian Grand Prix instead of *country grand prix* - ugly! Can do this by parsing Wikipedia's table at [this](https://en.wikipedia.org/wiki/2014_Formula_One_season) link.
- Add more metadata to the event somehow by looking up more attributes in the iCalendar VEVENT block [here](http://www.ietf.org/rfc/rfc2445.txt)
- A results as soon as they become available
