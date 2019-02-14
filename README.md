# CacheStrings
Specialized <hash,string> storage to ease your fight with all those UI-related memory leaks and GC bumps. You provide hash function, string-formatting method - all in one place (constructor call) - and the rest will be taken care of.
#
Use cases:
- Timers (!)
- Any kind of number UI fields where strings repeat (eventually) and pool size can be imagined as limited
#
```C#
using UnityEngine;
using UnityEngine.UI;
public class TestCacheIntString : MonoBehaviour
{
    public Text _displaySeconds;
    public Text _displayMinutes;
    public Text _displayHours;
    public Text _displayDays;
    public double _time = 0;
    CacheIntString cacheSeconds = new CacheIntString(
        (i)=>i%60 , //you describe how input seconds (key) will be translated to useful value (hash)
        (hash)=>hash.ToString("00") //you describe how string is built based on given value (hash)
        , 0 , 59 , 1 //initialization range and step, so cache will be warmed up and ready
    );
    CacheIntString cacheMinutes = new CacheIntString(
        (i)=>i/60%60 , // this translates input seconds to minutes
        (hash)=>hash.ToString("00") // this translates minute to string
        , 0 , 60 , 60 //minutes needs a step of 60 seconds
    );
    CacheIntString cacheHours = new CacheIntString(
        (i)=>i/(60*60)%24 , // this translates input seconds to hours
        (hash)=>hash.ToString("00") , // this translates hour to string
        0 , 24 , 60*60 //hours needs a step of 60*60 seconds
    );
    CacheIntString cacheDays = new CacheIntString(
        (i)=>i/(60*60*24) , // this translates input seconds to days
        (hash)=>hash.ToString() , // this translates day to string
        0 , 31 , 60*60*24 //days needs a step of 60*60*24 seconds
    );
    void Update ()
    {
        _time += Time.deltaTime;
        int seconds = Mathf.FloorToInt( (float)_time );
        _displaySeconds.text = cacheSeconds[ seconds ];
        _displayMinutes.text = cacheMinutes[ seconds ];
        _displayHours.text = cacheHours[ seconds ];
        _displayDays.text = cacheDays[ seconds ];
    }
}
```
#
