# Notes
### Js-note: Property Advantage in Jquery
```js
// Property code style
Object.defineProperty(window, 'more_actions_text', {
    get: function () {
        return $('#more-actions-btn').text();
    },
    set: function (v) {
        $('#more-actions-btn').text(v);
    }
});

/* Property code style (Arrow style)

 Object.defineProperty(window, 'more_actions_text', {
    get: () => $('#more-actions-btn').text(),
    set: (v) => $('#more-actions-btn').text(v)
 });
 */

more_actions_text = 1;

// Simple style througth jQuery
$('#more-actions-btn').text(1)
```

### PHP-note: use mongo cursor
```PHP
$alertCollection = MongoRecordPeer::collection('alert');

$cursor = $alertCollection->find([
    'callout_id' => [
        '$in' => $expiredOpportunitiesIds
    ]
]);

foreach ($cursor as $alertDocument) {
    $alertCollection->remove(['_id' => $alertDocument['_id']]);
    
    $removedCounter++;
}

// or short case ...

$alertCollection->remove([
    'callout_id' => [
        '$in' => $expiredOpportunitiesIds
    ]
], [
    'multiple' => true
])
```
### Jquery+Bootstrap note: Bootstrap highlighting of jQuery element
```js
/**
 * @param arg
 * @returns {jQuery}
 */
$.fn.highlightElement = function (arg) {

    $(this).off('focusin');

    var opts;

    if (arguments.length == 2
        &&
        typeof arguments[0] === 'string'
        &&
        typeof +arguments[1] === 'number'
    ) {

        opts = {
            type: arguments[0],
            delay: +arguments[1]
        }

    } else {

        switch (true) {
            case typeof arg === 'string':

                opts = { type: arg };
                break;

            case typeof +arg === 'number' && !isNaN(+arg):

                opts = { delay: arg };
                break;

            default:

                opts = arg || {};
        }
    }

    var glyphiconClassByTypeMap = {
            success: 'glyphicon-ok',
            warning: 'glyphicon-warning-sign',
            error: 'glyphicon-remove'
        };

    opts = $.extend({}, {

        type: 'error',
        $parent: $(this).parent(),
        feedback: true

    }, opts);

    $(this).next('.form-control-feedback').remove();
    opts.$parent
        .removeClass('has-feedback')
        .removeClass('has-error')
        .removeClass('has-warning')
        .removeClass('has-success')
    ;

    if (['error', 'warning', 'success'].indexOf(opts.type) !== -1) {

        if (opts.feedback) {
            opts.$parent
                .addClass('has-' + opts.type)
                .addClass('has-feedback')
                .append($('<span>', {
                    class: 'form-control-feedback glyphicon ' + glyphiconClassByTypeMap[opts.type]
                }));

            if (opts.type === 'error') {
                $(this).focus();
            }

        } else {

            opts.$parent.addClass('has-' + opts.type);
        }

        var timerId, hide = function () {

            if (opts.feedback) {
                opts.$parent
                    .removeClass('has-feedback')
                    .removeClass('has-' + opts.type)
                    .find('span.form-control-feedback').remove();

            } else {

                opts.$parent.removeClass('has-' + opts.type)
            }

            $(this).off('focusin');
        };

        hide = hide.bind(this);

        if (opts.delay) {

            timerId = setTimeout(hide, +opts.delay);
        }

        $(this).focusin(function () {

            if (timerId) {
                clearTimeout(timerId);
            }

            hide();
        });

    }

    return $(this);
};

```
#### Call example: 
```TWIG
{{ object.text|viewMore|raw }}
{# OR #}
{{ object.text|viewMore({limit: 70})|raw }}
{# OR without twig extension use #}
{% include '@AppBundles/MyTwigExtensions/textCollapse.twig' with {text: object.text } %}
```

### JS-note: Use private and public fields/methods
```TS
Object.defineProperty(this, 'factory', {
    get: () => {
        let privateField = 2;
        let privateFn = () => -1;
        return {
            pubField: 3,
            pubFn: function() {
                debugger
                return this.pubField + privateField;
            }
        };
    }
});
```
 
### JS-note (ES2016): student API (http://studentgo.ru/)
```JS
let studList = [];
$.post('http://studentgo.ru/search', {q: 'Яковенко' }).done((data) => {

    $(data).find('.student').each((_, studentBlock) => {

        let $studentBlock = $(studentBlock);
        let studentData = {
            studentId: $studentBlock.data('student-id'),
            studentName: $studentBlock.data('student-name')
        };

        $.post('http://studentgo.ru/student', studentData).done((data) => {

            let $studentDetailsBlock = $(data);
            let studentDetails = {
                fullname: $studentDetailsBlock.find('.name-student').html().replace(/\<br\>/g, ' '),
                group: {
                    id: $studentDetailsBlock.find('.groupName > a').data('group-id'),
                    name: $studentDetailsBlock.find('.groupName > a').data('group-name')
                },
                urls: $studentDetailsBlock.find('.social > a').map((_, link) => link.href ).toArray()
            };
            
            studList.push(studentDetails);
        });
    });
});
...
console.dir(studList);
```
### PHP-cache through closure (laravel example)
```php
// AppCacheService.php
namespace App\Services;

class AppCacheService
{
    /**
     * Minutes
     * @var int
     */
    private $cacheTtl = 1;

    /**
     * @var \Illuminate\Cache\Repository
     */
    private $cacheRepository;

    /**
     * AppCacheService constructor.
     */
    public function __construct()
    {
        $this->cacheRepository = \Cache::store(env('CACHE_DRIVER'));
    }

    /**
     * @param $key
     * @param \Closure|null $handler
     * @param null $ttl
     * @return mixed|null
     */
    public function get($key, \Closure $handler = null, $ttl = null)
    {
        if ($this->cacheRepository->has($key)) {

            $result = json_decode($this->cacheRepository->get($key));

        } else {

            if ($handler) {

                $result = $handler();
                $this->set($key, $result, $ttl);

            } else {

                $result = null;
            }
        }

        return $result;
    }

    /**
     * @param $key
     * @param $value
     * @param null $ttl
     */
    public function set($key, $value, $ttl = null)
    {
        $this->cacheRepository->add($key, json_encode($value), $ttl ? $ttl : $this->cacheTtl);
    }

    /**
     * @param $key
     */
    public function remove($key)
    {
        $this->cacheRepository->forget($key);
    }
}

// without cache (MyController.php)
// any handling with big data 
...
$objA = 1;
$objB = 2;

// use case (MyController.php)
list($objA, $objB) = $cache->get('my.cache.key', function() {
    // any handling with big data 
    ...
    $objA = 1;
    $objB = 2;

    return [$objA, $objB];
});

```
### Localstorage cache (Angularjs 2 example)
```ts
import {Injectable} from '@angular/core';
import {Observable} from 'rxjs/Rx';
import {Utils} from '../utils';

interface UnitStorage {
    data: any;
    expire_at: number;
}

@Injectable()
export class LocalStorageService {

    private support: boolean;

    constructor() {

        this.support = false;
        if (typeof localStorage !== 'undefined') {

            try {

                localStorage.setItem('test', 'test');
                localStorage.removeItem('test');

                this.support = true;

            } catch (err) {
            }
        }
    }

    cache(key: string, handler: (done: Function) => Promise<any>): Promise<any> {

        let cacheValue = this.get(key);
        return cacheValue ? Observable.of(cacheValue).toPromise() : handler((data: any, duration = null) => {
            this.set(key, data, duration);
            return data;
        });
    }

    get(key: string): any {

        let result: UnitStorage = null;

        if (this.support) {

            let value: UnitStorage = JSON.parse(localStorage.getItem(Utils.hex(key)));
            if (value) {
                if (value.expire_at) {

                    if ((new Date()).getTime() < value.expire_at) {

                        result = value.data;

                    } else {

                        this.remove(key);
                    }

                } else {

                    result = value.data;
                }
            }
        }

        return result;
    }

    set(key: string, value: any, duration: number = null) {

        if (this.support) {
            localStorage.setItem(Utils.hex(key), JSON.stringify({
                data: value,
                expire_at: duration ? (new Date()).getTime() + duration * 60 * 1000 : null
            }));
        }
    }

    remove(key) {

        if (this.support) {
            localStorage.removeItem(Utils.hex(key));
        }
    }
}


// use case
// without cache
getObjects(id: number): Promise<Array<MyObject>> {

    let url = `api/test?object_id=${ id }`;
    return this.http.get(url)
        .toPromise()
        .then(response => response.json() as Array<MyObject>)
        .catch(this.handleError));
}


// use cache
getObjects(id: number): Promise<Array<MyObject>> {

    let url = `api/test?object_id=${ id }`;
    return this.localStorageService.cache(url, keep => this.http.get(url)
        .toPromise()
        .then(response => keep(response.json() as Array<MyObject>))
        .catch(this.handleError));
}

// use expired cache
getObjects(id: number): Promise<Array<MyObject>> {

    let url = `api/test?object_id=${ id }`;
    return this.localStorageService.cache(url, keep => this.http.get(url)
        .toPromise()
        .then(response => keep(response.json() as Array<MyObject>, 60)) // 60 minutes
        .catch(this.handleError));
}

```

### C Sharp serialization EF-model by custom properties
```c#
using System.Collections.Generic;
using Newtonsoft.Json;
using System.Reflection;
using System.Collections;
...
public static string toJsonByCustomProperties(object obj, List<object> properties)
{
    return toJson(serializeToDictionary(obj, properties));
}

public static Dictionary<string, object> serializeToDictionary(object obj, List<object> properties)
{
    Dictionary<string, object> result = new Dictionary<string, object>();

    foreach (var property in properties)
    {
        if (property.GetType() == typeof(string))
        {
            string key = (string)property;
            var value = obj.GetType().GetProperty(key).GetValue(obj, null);
            result.Add(key, value);
        }
        else
        {
            Type instance = property.GetType();

            string key = (string)instance
                .GetProperty("key")
                .GetValue(property);

            object childObject = obj.GetType()
                    .GetProperty(key)
                    .GetValue(obj);

            List<object> childProperties = (List<object>)instance
                    .GetProperty("properties")
                    .GetValue(property);

            if (childObject != null)
            {
                PropertyInfo propertyInfo = instance.GetProperty("type");
                string type = propertyInfo != null ? (string)propertyInfo.GetValue(property, null) : null;
                if (type == null || type == "object")
                {
                    result.Add(key, serializeToDictionary(childObject, childProperties));
                }
                else
                {
                    List<object> list = new List<object>();
                    foreach (var item in (IEnumerable)childObject)
                    {
                        list.Add(serializeToDictionary(item, childProperties));
                    }

                    result.Add(key, list);
                }
            }
            else
            {
                result.Add(key, null);
            }
        }
    }

    return result;
}

// use case
private List<object> calloutProperties = new List<object>()
{
    "id",
    "created_at",
    new
    {
        key = "airtickets",
        type = "array",
        properties = new List<object>()
        {
            "id",
            "departure_at",
            "payment",
            new
            {
                key = "flight",
                properties = new List<object>
                {
                    "id",
                    "code",
                    "flight_at",
                    "duration"
                }
            }
        }
    }
};

[HttpGet]
public JsonResult PredefinedCallouts()
{
    Dictionary<string, object> result = new Dictionary<string, object>();

    IQueryable<callout> qb = _dbContext.callouts
        .Where(c => c.is_predefined == 1);

    result.Add("callouts", new List<object>());

    int pageSize = int.Parse(Request["pageSize"]);
    int pageNumber = int.Parse(Request["pageNumber"]);
    int offset = (pageNumber - 1) * pageSize;

    List<callout> callouts = qb
        .OrderByDescending(c => c.created_at)
        .Skip(offset)
        .ToList();

    foreach (callout c in callouts)
    {
        ((List<object>)result["callouts"]).Add(Utils.toJsonByCustomProperties(c, calloutProperties));
    }
    
    return Json(result, JsonRequestBehavior.AllowGet);
}
```
