# QWeather SDK v4 Android 开发文档 (已弃用)

- 最后版本: 4.20
- 停服日期: 2026-09-01
- 替代产品: SDK v5+

## 配置

### 适配版本 {#os-requirement}

Android 4.4+

### 工程配置 {#project-configuration}

1. 解压文件，将文件夹内jar放入您的工程，并且引用
2. 配置Android Manifest 添加权限

**权限列表**

权限说明 | 代码
--------- | -------------
允许连接网络 | android.permission.INTERNET


**引用库**

```
    compile 'com.squareup.okhttp3:okhttp:3.12.12' （3.12.12+）
    compile 'com.google.code.gson:gson:2.6.2'   (2.6.2+)
```

**混淆**

请在您的混淆文件proguard-rules.pro中加入如下代码
请注意您引用的版本

```java
//  排除okhttp
 -dontwarn com.squareup.**
 -dontwarn okio.**
 -keep public class org.codehaus.* { *; }
 -keep public class java.nio.* { *; }

//  排除QWeather
 -dontwarn com.qweather.sdk.**
 -keep class com.qweather.sdk.** { *;}
```
 
### 数据访问配置 {#data-access-configuration}

**日志功能**

SDK 不再提供日志功能， 错误信息可由回调函数 OnError 中的 Throwable 对象提供

**账户初始化**

使用 SDK 时，需提前进行账户初始化（全局执行一次即可）

```java
HeConfig.init("PublicId", "PrivateKey");
```

**设置订阅版本**

默认为标准订阅，如使用免费订阅，可通过以下方法进行调整（全局执行一次即可）
 
```java
//切换至免费订阅
HeConfig.switchToDevService();
//切换至标准订阅
HeConfig.switchToBizService();
```

**数据访问示例**

根据您的需求调用不同的方法，接口回调方法中的参数就是接口返回的数据类

```java
/**
 * 实况天气数据
 * @param location 所查询的地区，可通过该地区ID、经纬度进行查询经纬度格式：经度,纬度
 *                 （英文,分隔，十进制格式，北纬东经为正，南纬西经为负)
 * @param lang     (选填)多语言，可以不使用该参数，默认为简体中文
 * @param unit     (选填)单位选择，公制（m）或英制（i），默认为公制单位
 * @param listener 网络访问结果回调
 */

QWeather.getWeatherNow(MainActivity.this, "101010300", Lang.ZH_HANS, Unit.METRIC, new QWeather.OnResultWeatherNowListener() {
    @Override
    public void onError(Throwable e) {
        Log.i(TAG, "getWeather onError: " + e);
    }

    @Override
    public void onSuccess(WeatherNowBean weatherBean) {
        Log.i(TAG, "getWeather onSuccess: " + new Gson().toJson(weatherBean));
        //先判断返回的status是否正确，当status正确时获取数据，若status不正确，可查看status对应的Code值找到原因
        if (Code.OK == weatherBean.getCode()) {
            WeatherNowBean.NowBaseBean now = weatherBean.getNow();
        } else {
            //在此查看返回数据失败的原因
            Code code = weatherBean.getCode();
            Log.i(TAG, "failed code: " + code);
        }
    }
});
```


## 城市搜索

城市搜索，可获取到该城市的基本信息，包括城市的Location ID（你需要这个ID去查询天气），多语言名称、经纬度、时区、海拔、Rank值、归属上级行政区域、所在行政区域等。 另外，城市搜索也可以帮助你在你的APP中实现模糊搜索，用户只需要输入1-2个字即可获得结果。

| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getGeoCityLookup| 城市查询  | GeoBean |

### 接口参数说明

* `location`(必选)需要查询地区的名称，支持文字、以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位）、[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或[Adcode](https://dev.qweather.com/docs/resource/glossary/#adcode)（仅限中国城市）。例如 `location=北京` 或 `location=116.41,39.92`
* `adm`城市的上级行政区划，可设定只在某个行政区划范围内进行搜索，用于排除重名城市或对结果进行过滤。例如 `adm=beijing`
* `range`搜索范围，可设定只在某个国家或地区范围内进行搜索，国家和地区名称需使用[ISO 3166 所定义的国家代码](https://dev.qweather.com/docs/resource/glossary/#iso-3166)。如果不设置此参数，搜索范围将在所有城市。例如 `range=cn`
* `number`返回结果的数量，取值范围1-20，默认返回10个结果。
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。


### 示例代码

```java
QWeather.getGeoCityLookup(Context context, String location, String adm, Range range, int number, Lang lang, final QWeather.OnResultGeoListener listener);

QWeather.getGeoCityLookup(Context context, String location, Range range, int number, Lang lang, final QWeather.OnResultGeoListener listener);

QWeather.getGeoCityLookup(Context context, String location, int number, Lang lang, final QWeather.OnResultGeoBeansListener listener);

QWeather.getGeoCityLookup(Context context, String location, final QWeather.OnResultGeoBeansListener listener);
```

### GeoBean属性

| 属性 | 说明 | 示例值 |
|--|--|--|
| getCode | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/) | 200 |
| getLocationBean | 城市数据 | List&lt;LocationBean&gt; |


**Refer**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getSourcesList | 原始数据来源 | QWeather |
| getLicenseList | 使用许可 | QWeather Developers License |


**LocationBean 基础信息**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getName | 地区／城市名称 | 卓资 |
| getId | 地区／城市ID | 101080402 |
| getLon | 地区／城市经度 | 112.577702 |
| getLat | 地区／城市纬度 | 40.89576 |
| getAdm2 | 该地区／城市的上级城市 | 乌兰察布 |
| getAdm1 | 该地区／城市所属行政区域 | 内蒙古 |
| getCountry | 该地区／城市所属国家名称 | 中国 |
| getTz | 该地区／城市所在时区 | Asia/Shanghai |
| getUtcOffset | 该地区/城市目前与UTC时间偏移的小时数 | +08:00 |
| getIsDst | 该地区/城市是否当前处于夏令时,1: 表示当前处于夏令时, 0: 表示当前不是夏令时 | 0 |
| getType | 该地区／城市的属性 | city |
| getRank | 该地区／城市评分 | 10 |
| getFxLink | 城市的天气预报网页链接 | https://www.qweather.com/weather/zhuozi-101080402.html |

## POI搜索

使用关键字和坐标查询POI信息（景点、火车站、飞机场、港口等）

| 接口代码| 接口说明            | 数据类     |
| ----------- | --------------- | ---------- |
| getGeoPoiLookup| POI搜索  | GeoPoiBean |

### 接口参数说明

* `location`(必选)需要查询地区的名称，支持文字、以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位）、[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或[Adcode](https://dev.qweather.com/docs/resource/glossary/#adcode)（仅限中国城市）。例如 `location=北京` 或 `location=116.41,39.92`
* `type`(必选)POI类型，可选择搜索某一类型的POI。
  * `scenic` 景点
  * `CSTA` 潮流站点
  * `TSTA` 潮汐站点
* `city`选择POI所在城市，可设定只搜索在特定城市内的POI信息。城市名称可以是文字或城市的LocationID，城市名称为精准匹配，建议使用LocaitonID，如文字无法匹配，则数据返回为空。**默认不限制特定城市**。
* `number`返回结果的数量，取值范围1-20，默认返回10个结果。
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java
QWeather.getGeoPoiLookup(Context context, String location, String city, int number, Type type, Lang lang, final OnResultGeoPoiListener listener);

QWeather.getGeoPoiLookup(Context context, String location, Type type, final QWeather.OnResultGeoPoiListener listener);
```

## POI范围搜索

提供指定区域范围内查询所有POI信息。

| 接口代码| 接口说明           | 数据类     |
| ----------- | -------------- | ---------- |
| getGeoPoiRange| POI范围搜索  | GeoPoiBean |

### 接口参数说明

* `location`(必选)需要查询地区的以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位）。例如 `location=116.41,39.92`
* `type`(必选)POI类型，可选择搜索某一类型的POI。
  * `scenic` 景点
  * `CSTA` 潮流站点
  * `TSTA` 潮汐站点
* `radius`搜索范围，可设置搜索半径，取值范围1-50，单位：公里。**默认5公里**。
* `number`返回结果的数量，取值范围1-20，默认返回10个结果。
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。


### 示例代码

```java
QWeather.getGeoPoiRange(Context context, String location, int radius, int number, Type type, Lang lang, final OnResultGeoPoiListener listener);

QWeather.getGeoPoiRange(Context context, String location, int number, Type type, Lang lang, final OnResultGeoPoiListener listener);
```

## 实况天气

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。
* `unit`数据单位设置，可选值包括`unit=m`（公制单位，默认）和`unit=i`（英制单位）。更多选项和说明参考[度量衡单位](https://dev.qweather.com/docs/resource/unit)。


### 示例代码

```java
QWeather.getWeatherNow(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherNowListener listener) ;

QWeather.getWeatherNow(Context context, String location, QWeather.OnResultWeatherNowListener listener);
```
### WeatherNowBean属性

| 属性 | 说明 | 示例值 |
|--|--|--|
| getCode | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/) | 200 |
| getNow | NowBean 实况天气 | NowBaseBean |
| getRefer | Refer 数据来源以及数据授权 | Refer |
| getBasic | Basic 基础信息 | Basic |

**Refer**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getSourcesList | 原始数据来源 | QWeather |
| getLicenseList | 使用许可 | QWeather Developers License |

**Basic**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00 |
| getFxLink | 所查询城市的天气预报网页 | https://www.qweather.com/weather/beijing-101010100.html |

**NowBaseBean 实况天气**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getObsTime | 实况观测时间 | 2013-12-30T13:14+08:00 |
| getFeelsLike | 体感温度，默认单位：摄氏度 | 23 |
| getTemp | 温度，默认单位：摄氏度 | 21 |
| getIcon | 天气状况的[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/) | 100 |
| getText | 天气状况的文字描述，包括阴晴雨雪等天气状态的描述 | 晴 |
| getWind360 | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)360角度 | 305 |
| getWindDir | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction) | 西北 |
| getWindScale | [风力等级](https://dev.qweather.com/docs/resource/wind-info/#wind-scale) | 3-4 |
| getWindSpeed | [风速](https://dev.qweather.com/docs/resource/wind-info/#wind-speed)，公里/小时 | 15 |
| getHumidity | 相对湿度 | 40 |
| getPrecip | 降水量 | 0 |
| getPressure | 大气压强 | 1020 |
| getVis | 能见度，默认单位：公里 | 10 |
| getCloud | 云量 | 23 |
| getDew | 露点温度 | -1 |

## 每日天气预报

提供全球城市未来3 - 30天天气预报，包括：日出日落、月升月落、最高最低温度、天气白天和夜间状况、风力、风速、风向、相对湿度、大气压强、降水量、降水概率、露点温度、紫外线强度、能见度等。

| 接口代码| 接口说明               | 数据类           |
| ---------------- | ------------- | ---------------- |
| getWeather3D| 3天预报天气数据    | WeatherDailyBean |
| getWeather7D| 7天预报天气数据    | WeatherDailyBean |
| getWeather10D| 10天预报天气数据  | WeatherDailyBean |
| getWeather15D| 15天预报天气数据  | WeatherDailyBean |
| getWeather30D| 30天预报天气数据  | WeatherDailyBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。
* `unit`数据单位设置，可选值包括`unit=m`（公制单位，默认）和`unit=i`（英制单位）。更多选项和说明参考[度量衡单位](https://dev.qweather.com/docs/resource/unit)。

### 示例代码

```java
/**
 * 获取3天预报数据
 */
QWeather.getWeather3D(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherDailyListener listener) ;

QWeather.getWeather3D(Context context, String location, QWeather.OnResultWeatherDailyListener listener);

/**
 * 获取7天预报数据
 */
QWeather.getWeather7D(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherDailyListener listener) ;

QWeather.getWeather7D(Context context, String location, QWeather.OnResultWeatherDailyListener listener);

/**
 * 获取10天预报数据
 */
QWeather.getWeather10D(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherDailyListener listener) ;

QWeather.getWeather10D(Context context, String location, QWeather.OnResultWeatherDailyListener listener);

/**
 * 获取15天预报数据
 */
QWeather.getWeather15D(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherDailyListener listener) ;

QWeather.getWeather15D(Context context, String location, QWeather.OnResultWeatherDailyListener listener);

/**
 * 获取30天预报数据
 */
QWeather.getWeather30D(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherDailyListener listener) ;

QWeather.getWeather30D(Context context, String location, QWeather.OnResultWeatherDailyListener listener);
```

### WeatherDailyBean属性

| 属性 | 说明 | 示例值 |
|--|--|--|
| getCode | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/) | 200 |
| getDaily | DailyBean 逐天天气 | List&lt;DailyBean&gt; |
| getRefer | Refer 数据来源以及数据授权 | Refer |
| getBasic | Basic 基础信息 | Basic |

**Refer**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getSourcesList | 原始数据来源 | QWeather |
| getLicenseList | 使用许可 | QWeather Developers License |

**Basic**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00 |
| getFxLink | 所查询城市的天气预报网页 | https://www.qweather.com/weather/beijing-101010100.html |

**DailyBean 天气预报**

| 属性 | 说明 | 示例值 |
|--|--|--|
| getFxDate | 预报日期 | 2013-12-30 |
| getSunrise | [日出时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空** | 07:36 |
| getSunset | [日落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空** | 16:58 |
| getMoonRise | 当天[月升时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空** | 04:47 |
| getMoonSet | 当天[月落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空** | 14:59 |
| getMoonPhase | [月相名称](https://dev.qweather.com/docs/resource/sun-moon-info/#moon-phase) | 满月 |
| getMoonPhaseIcon | 月相[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/) | 804 |
| getTempMax | 最高温度 | 4 |
| getTempMin | 最低温度 | -5 |
| getIconDay | 白天天气状况的[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/) | 100 |
| getIconNight | 夜间天气状况的[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/) | 150 |
| getTextDay | 白天天气状况文字描述，包括阴晴雨雪等 | 晴 |
| getTextNight | 晚间天气状况文字描述，包括阴晴雨雪等 | 晴 |
| getWind360Day | 白天[风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)360角度 | 310 |
| getWind360Night | 夜间[风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)360角度 | 310 |
| getWindDirDay | 白天[风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction) | 西北风 |
| getWindDirNight | 夜间[风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction) | 西北风 |
| getWindScaleDay | 白天[风力等级](https://dev.qweather.com/docs/resource/wind-info/#wind-scale) | 1-2 |
| getWindScaleNight | 夜间[风力等级](https://dev.qweather.com/docs/resource/wind-info/#wind-scale) | 1-2 |
| getWindSpeedDay | 白天[风速](https://dev.qweather.com/docs/resource/wind-info/#wind-speed)，公里/小时 | 14 |
| getWindSpeedNight | 夜间[风速](https://dev.qweather.com/docs/resource/wind-info/#wind-speed)，公里/小时 | 14 |
| getHumidity | 相对湿度 | 37 |
| getPrecip | 降水量 | 0.0 |
| getPressure | 大气压强 | 1018 |
| getCloud | 当天云量 | 23 |
| getUvIndex | 紫外线强度指数 | 3 |
| getVis | 能见度，单位：公里 | 10 |

## 逐小时天气预报

逐小时天气预报Android SDK，提供全球城市未来24-168小时逐小时天气预报，包括：温度、天气状况、风力、风速、风向、相对湿度、大气压强、降水概率、露点温度、云量。

| 接口代码| 接口说明                   | 数据类            |
| ------------------- | -------------- | ----------------- |
| getWeather24H| 24小时预报天气数据    | WeatherHourlyBean |
| getWeather72H| 72小时预报天气数据    | WeatherHourlyBean |
| getWeather168H| 168小时预报天气数据  | WeatherHourlyBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。
* `unit`数据单位设置，可选值包括`unit=m`（公制单位，默认）和`unit=i`（英制单位）。更多选项和说明参考[度量衡单位](https://dev.qweather.com/docs/resource/unit)。

### 示例代码

```java
/**
 * 获取24小时预报数据
 */
QWeather.getWeather24Hourly(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherHourlyListener listener);

QWeather.getWeather24Hourly(Context context, String location, QWeather.OnResultWeatherHourlyListener listener);

/**
 * 获取72小时预报数据
 */
QWeather.getWeather72Hourly(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherHourlyListener listener) ;

QWeather.getWeather72Hourly(Context context, String location, QWeather.OnResultWeatherHourlyListener listener);

/**
 * 获取168小时预报数据
 */
QWeather.getWeather168Hourly(Context context, String location, Lang lang, Unit unit, QWeather.OnResultWeatherHourlyListener listener) ;

QWeather.getWeather168Hourly(Context context, String location, QWeather.OnResultWeatherHourlyListener listener);

```

### WeatherHourlyBean属性

| 属性      | 说明                       | 示例值                 |
| --------- | -------------------------- | ---------------------- |
| getCode   | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200 |
| getHourly | HourlyBean 逐小时天气      | List&lt;HourlyBean&gt; |
| getRefer  | Refer 数据来源以及数据授权 | Refer                  |
| getBasic  | Basic 基础信息             | Basic                  |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/weather/beijing-101010100.html |

**HourlyBean 逐小时天气**

| 属性         | 说明                                     | 示例值           |
| ------------ | ---------------------------------------- | ---------------- |
| getFxTime    | 预报时间           | 2013-12-30T13:00+08:00 |
| getTemp      | 温度                                     | 2                |
| getIcon      | 天气状况的[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/)                             | 101              |
| getText      | 气状况的文字描述，包括阴晴雨雪等                             | 多云             |
| getWind360   | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)360角度                              | 290              |
| getWindDir   | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)                                     | 西北             |
| getWindScale | [风力等级](https://dev.qweather.com/docs/resource/wind-info/#wind-scale)                                     | 3-4              |
| getWindSpeed | [风速](https://dev.qweather.com/docs/resource/wind-info/#wind-speed)，公里/小时                          | 15               |
| getHumidity  | 相对湿度                                 | 30               |
| getPop       | 逐小时预报降水概率，百分比数值，可能为空 | 5                |
| getPrecip    | 逐小时预报降水量，默认单位：毫米         | 1.2              |
| getPressure  | 大气压强                                 | 1030             |
| getCloud     | 云量，百分比                             | 15               |
| getDew       | 露点温度                                 | 5                |

## 天气灾害预警

天气灾害预警Android SDK可以获取中国及全球多个国家或地区官方发布的实时天气灾害预警数据。

> **提示：**关于更多天气预警数据的说明，请参考[实用资料-预警信息](https://dev.qweather.com/docs/resource/warning-info/)。

| 接口代码| 接口说明        | 数据类      |
| ------------ | ---------- | ----------- |
| getWarning| 天气灾害预警  | WarningBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java
QWeather.getWarning(Context context, String location, final QWeather.OnResultWarningListener listener) ;

QWeather.getWarning(Context context, String location, Lang lang, final QWeather.OnResultWarningListener listener) ;

```

### WarningBean属性

| 属性            | 说明                       | 示例值                      |
| --------------- | -------------------------- | --------------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200  |
| getBeanBaseList | 灾害预警                   | List&lt;WarningBeanBase&gt; |
| getRefer        | Refer 数据来源以及数据授权 | Refer                       |
| getBasic        | Basic 基础信息             | Basic                       |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/severe-weather/beijing-101010100.html |

**WarningBeanBase 预警信息**

| 属性         | 说明                               | 示例值                                                           |
| ------------ | ---------------------------------- | ---------------------------------------------------------------- |
| getId        | 预警id                             | 10102010020210514101500714183119
| getPubTime   | 预警发布时间   | 2017-10-25T12:03+08:00                                                 |
| getTitle     | 预警信息标题                       | 广东省深圳市气象台发布雷电黄色预警                               |
| getSender | 预警发布单位，可能为空 |深圳市气象台 |
| getStartTime | 预警开始时间                       | 2017-10-25T12:03+08:00                                |
| getEndTime   | 预警结束时间                       | 2017-10-25T12:03+08:00                                |
| getStatus    | 预警状态                           | 预警中                                                           |
| getSeverity     | [预警严重等级](https://dev.qweather.com/docs/resource/warning-info/#severity)               | Moderate                                                             |
| getSeverityColor     | [预警严重等级颜色](https://dev.qweather.com/docs/resource/warning-info/#severity-color)，**可能为空**               | Blue                                                             |
| getType      | [预警类型ID](https://dev.qweather.com/docs/resource/warning-info/#warning-type)     | 1014                                                             |
| getTypeName      | [预警类型名称](https://dev.qweather.com/docs/resource/warning-info/#warning-type)     | 雷电                                                             |
| getUrgency      | [预警信息的紧迫程度](https://dev.qweather.com/docs/resource/warning-info/#urgency)，**可能为空**     | Immediate                                                             |
| getCertainty      | [预警信息的确定性](https://dev.qweather.com/docs/resource/warning-info/#certainty)，**可能为空**     | Likely                                                             |
| getText      | 预警详细信息                       | 深圳市气象局于10月04日12时59分发布雷电黄色预警信号，请注意防御。 |
| getRelated      | 与本条预警相关联的预警ID，当预警状态为cancel或update时返回。可能为空 | 10102010020210513101500714846231 |

## 天气指数预报

获取中国和海外城市天气生活指数预报数据。

查看生活指数类型和等级 <https://dev.qweather.com/docs/resource/indices-info/>

| 接口代码| 接口说明         | 数据类      |
| ----------- | ------------ | ----------- |
| getIndices1D| 1天生活指数  | IndicesBean |
| getIndices3D| 3天生活指数  | IndicesBean |

### 请求参数

请求参数包括必选和可选参数，如不填写可选参数将使用其默认值。

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `IndicesType`(必选)生活指数的类型，包括洗车指数、穿衣指数、钓鱼指数等。可以一次性获取多个类型的生活指数，多个类型请添加进同一数组。各项生活指数并非适用于所有城市。具体生活指数的ID和等级参考[天气指数信息](https://dev.qweather.com/docs/resource/indices-info/)。


### 示例代码

```java
/**
 * 获取1天生活指数数据
 */
QWeather.get1DIndices(Context context, String location, Lang lang, List<IndicesType> types, QWeather.OnResultIndicesListener listener);

/**
 * 获取3天生活指数数据
 */
QWeather.get3DIndices(Context context, String location, Lang lang, List<IndicesType> types, QWeather.OnResultIndicesListener listener) ;
```

### IndicesBean属性

| 属性         | 说明                       | 示例值                |
| ------------ | -------------------------- | --------------------- |
| getCode      | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200 |
| getDailyList | 生活指数逐天预报数据       | List&lt;DailyBean&gt; |
| getRefer     | Refer 数据来源以及数据授权 | Refer                 |
| getBasic     | Basic 基础信息             | Basic                 |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/indices/beijing-101010100.html |


**DailyBean 当天生活指数**

| 属性        | 说明         |
| ----------- | ----------------------- |
| getDate     | 预报日期，格式 yyyy-MM-dd     |
| getLevel    | 生活指数预报等级          |
| getCategory | 生活指数预报级别名称     |
| getName     | 生活指数名称             |
| getType     | 生活指数类型 |
| getText     | 生活指数详细描述         |


## 分钟级降水

分钟级降水Android SDK（临近预报）支持中国1公里精度的未来2小时每5分钟降雨预报数据。

| 接口代码| 接口说明       | 数据类       |
| ---------- | ----------- | ------------ |
| getMinutely | 分钟级降雨 | MinutelyBean |

### 接口参数说明

* `location`(必选)需要查询地区的以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位）。例如 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java
QWeather.getMinutely(Context context, double longitude, double latitude, QWeather.OnResultMinutelyListener listener);

QWeather.getMinutely(Context context, double longitude, double latitude, Lang lang, QWeather.OnResultMinutelyListener listener);
```

### GridMinutelyBean属性

| 属性            | 说明                       | 示例值               |
| --------------- | -------------------------- | -------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200  |
| getSummary      | 分钟降水描述               | 未来2小时无降雨      |
| getMinutelyList | 临近预报                   | List&lt;Minutely&gt; |
| getRefer        | Refer 数据来源以及数据授权 | Refer                |
| getBasic        | Basic 基础信息             | Basic                |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com |

**Minutely 未来两小时5分钟降水量**

| 属性      | 说明                       | 示例值           |
| --------- | -------------------------- | ---------------- |
| getFxTime | 时间 | 2013-12-30T20:35+08:00 |
| getPrecip | 5分钟累计降水量，单位毫米                     | 10               |
| getType   | 降水类型                   | rain             |

## 实时空气质量

实时空气质量Android SDK，支持中国3000+市县区以及1700+国控站点实时空气质量的查询，包括AQI数值、空气质量等级、首要污染物、PM10、PM2.5、臭氧、二氧化氮、二氧化硫、一氧化碳数值。

| 接口代码| 接口说明           | 数据类     |
| ---------------- | --------- | ---------- |
| getAirNow| 空气质量实况数据  | AirNowBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java
QWeather.getAirNow(Context context, String location, Lang lang, QWeather.OnResultAirNowListener listener)
```

### AirNowBean 属性

| 属性                 | 说明                       | 示例值                        |
| -------------------- | -------------------------- | ----------------------------- |
| getCode              | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200    |
| getAirNowStationBean | AQI站点实况                | List&lt;AirNowStationBean&gt; |
| getNow               | AQI城市实况                | NowBean                       |
| getRefer             | Refer 数据来源以及数据授权 | Refer                         |
| getBasic             | Basic 基础信息             | Basic                         |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/air/beijing-101010100.html |

**NowBean AQI城市实况**

| 属性        | 说明                              | 示例值           |
| ----------- | --------------------------------- | ---------------- |
| getPubTime  | 数据发布时间 | 2017-03-20T12:30+08:00 |
| getAqi      | 空气质量指数，AQI和PM25的关系     | 74               |
| getPrimary  | 主要污染物                        | PM2.5             |
| getLevel    | 实时空气质量指数等级              | 2                |
| getCategory | 实时空气质量指数级别              | 良               |
| getPm10     | pm10                              | 78               |
| getPm2p5    | pm2.5                              | 66               |
| getNo2      | 二氧化氮                          | 40               |
| getSo2      | 二氧化硫                          | 30               |
| getCo       | 一氧化碳                          | 0.3               |
| getO3       | 臭氧                              | 20               |

**AirNowStationBean AQI站点实况**

| 属性        | 说明                              | 示例值           |
| ----------- | --------------------------------- | ---------------- |
| getPubTime  | 数据发布时间 | 2017-03-20T12:30+08:00 |
| getName     | 站点名称                          | 万寿西宫         |
| getId       | 站点ID，请参考所有站点ID          | CNA1001          |
| getAqi      | 空气质量指数，AQI和PM25的关系     | 74               |
| getPrimary  | 主要污染物                        | PM2.5             |
| getLevel    | 实时空气质量指数等级              | 2                |
| getCategory | 实时空气质量指数级别              | 良               |
| getPm10     | pm10                              | 78               |
| getPm2p5    | pm25                              | 66               |
| getNo2      | 二氧化氮                          | 40               |
| getSo2      | 二氧化硫                          | 30               |
| getCo       | 一氧化碳                          | 0.3               |
| getO3       | 臭氧                              | 20               |

## 空气质量预报

空气质量每日预报Android SDK，支持全国3000+市县区空气质量预报数据的查询，包括AQI预报、首要污染物预报、空气质量等级预报。

| 接口代码| 接口说明             | 数据类       |
| ------------------- | -------- | ------------ |
| getAir5D| 空气质量5天预报数据  | AirDailyBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java

/**
 * 空气质量5天预报数据
 */

QWeather.getAir5D(Context context, String location, Lang lang, QWeather.OnResultAirDailyListener listener)

```

### AirDailyBean 属性

| 属性        | 说明                       | 示例值                |
| ----------- | -------------------------- | --------------------- |
| getCode     | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200  |
| getAirDaily | 空气质量 AQI 5天预报       | List&lt;DailyBean&gt; |
| getRefer    | Refer 数据来源以及数据授权 | Refer                 |
| getBasic    | Basic 基础信息             | Basic                 |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/air/beijing-101010100.html |

**DailyBean AQI城市逐天预报**

| 属性        | 说明                          | 示例值     |
| ----------- | ----------------------------- | ---------- |
| getFxDate   | 预报日期,格式yyyy-MM-dd       | 2017-08-09 |
| getAqi      | 空气质量指数，AQI和PM25的关系 | 74         |
| getPrimary  | 主要污染物                    | PM2.5       |
| getLevel    | 实时空气质量指数等级          | 2          |
| getCategory | 实时空气质量指数级别          | 良         |

## 天气时光机

获取最近10天的天气历史再分析数据。

> 例如今天是12月30日，最多可获取12月20日至12月29日的天气历史数据。

| 接口代码| 接口说明                  | 数据类             |
| ------------ | -------------------- | ------------------ |
| getWeatherHistorical| 历史天气数据  | HistoryWeatherBean |

### 接口参数说明

* `location`(必选)需要查询的地区，仅支持[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)，LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100`
* `date`(必选)选择日期，最多可选择最近10天（不包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。
* `unit`数据单位设置，可选值包括`unit=m`（公制单位，默认）和`unit=i`（英制单位）。更多选项和说明参考[度量衡单位](https://dev.qweather.com/docs/resource/unit)。


### 示例代码

```java
QWeather.getHistoricalWeather(Context context, String location, String date, QWeather.OnResultWeatherHistoricalBeanListener listener) ;

QWeather.getHistoricalWeather(Context context, String location, String date, Lang lang, Unit unit, QWeather.OnResultWeatherHistoricalBeanListener listener)
```

### HistoryWeatherBean属性

| 属性           | 说明                       | 示例值                 |
| -------------- | -------------------------- | ---------------------- |
| getCode        | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200|
| getDailyBean   | 当天概况                   | DailyBean              |
| getHourlyBeans | 当天逐小时数据             | List&lt;HourlyBean&gt; |
| getRefer       | Refer 数据来源以及数据授权 | Refer                  |
| getBasic       | Basic 基础信息             | Basic                  |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather     |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/historical/beijing-101010100.html |

**DailyBean 基础信息**

| 属性         | 说明     | 示例值     |
| ------------ | -------- | ---------- |
| getDate      | 预报日期 | 2013-12-30 |
| getSunrise   | [日出时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空** | 07:36      |
| getSunset    | [日落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空** | 16:58      |
| getMoonRise  | 当天[月升时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空** | 04:47      |
| getMoonSet   | 当天[月落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空** | 14:59      |
| getMoonPhase | 月相     | 满月       |
| getTempMax   | 最高温度 | 4          |
| getTempMin   | 最低温度 | -5         |
| getHumidity  | 相对湿度 | 37         |
| getPrecip    | 降水量   | 0          |
| getPressure  | 大气压强 | 1018       |

**HourlyBean 基础信息**

| 属性         | 说明                                   | 示例值           |
| ------------ | -------------------------------------- | ---------------- |
| getTime      | 历史当天天气时间                        | 2013-12-30T13:00+08:00 |
| getTemp      | 温度                                   | 2                |
| getIcon      | 当天每小时天气状况的[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/)                           | 101              |
| getText      | 当天每小时天气状况的文字描述，包括阴晴雨雪等                           | 多云             |
| getWind360   | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)360角度                            | 290              |
| getWindDir   | [风向](https://dev.qweather.com/docs/resource/wind-info/#wind-direction)                                   | 西北             |
| getWindScale | [风力等级](https://dev.qweather.com/docs/resource/wind-info/#wind-scale)                                   | 3-4              |
| getWindSpeed | [风速](https://dev.qweather.com/docs/resource/wind-info/#wind-speed)                                   | 15               |
| getHumidity  | 湿度                                   | 30               |
| getPressure  | 大气压强                               | 1030             |
| getPrecip    | 逐小时预报降水量，默认单位：毫米       | 1.2              |

## 历史空气质量

获取最近10天的中国空气质量历史再分析数据。

> 例如今天是12月30日，最多可获取12月20日至12月29日的空气质量历史数据。


| 接口代码| 接口说明                  | 数据类            |
| ---------------- | ---------------- | ----------------- |
| getHistoricalAir| 历史空气质量数据  | HistoricalAirBean |

### 接口参数说明

* `location`(必选)需要查询的地区，仅支持[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)，LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100`
* `date`(必选)选择日期，最多可选择最近10天（不包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`
* `lang`多语言设置，请阅读[多语言](https://dev.qweather.com/docs/resource/language/)文档，了解我们的多语言是如何工作、如何设置以及数据是否支持多语言。

### 示例代码

```java
QWeather.getHistoricalAir(Context context, String location, String date, QWeather.OnResultAirHistoricalBeanListener listener) ;

QWeather.getHistoricalAir(Context context, String location, String date, Lang lang, Unit unit,QWeather.OnResultAirHistoricalBeanListener listener)
```

### HistoricalAirBean属性

| 属性              | 说明                       | 示例值                    |
| ----------------- | -------------------------- | ------------------------- |
| getCode           | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200   |
| getRefer          | Refer 数据来源以及数据授权 | Refer                     |
| getBasic          | Basic 基础信息             | Basic                     |
| getAirHourlyBeans | 当天逐小时空气质量数据     | List&lt;AirHourlyBean&gt; |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34+08:00     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/historical/beijing-101010100.html |

**AirHourlyBean 历史当天逐小时空气质量数据**

| 属性        | 说明                              | 示例值           |
| ----------- | --------------------------------- | ---------------- |
| getPubTime  | 数据发布时间 | 2017-03-20T12:30+08:00 |
| getAqi      | 空气质量指数，AQI和PM25的关系     | 74               |
| getPrimary  | 主要污染物                        | PM2.5             |
| getLevel    | 实时空气质量指数等级              | 2                |
| getCategory | 实时空气质量指数级别              | 良               |
| getPm10     | pm10                              | 78               |
| getPm2p5    | pm2.5                              | 66               |
| getNo2      | 二氧化氮                          | 40               |
| getSo2      | 二氧化硫                          | 30               |
| getCo       | 一氧化碳                          | 0.3               |
| getO3       | 臭氧                              | 20               |

## 潮汐

未来10天全球潮汐数据，包括满潮、干潮高度和时间，逐小时潮汐数据。

| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getOceanTide| 潮汐数据  | TideBean |

### 接口参数说明

* `location`(必选)需要查询的潮汐站点，请填写潮汐站点的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)，LocationID可通过[POI搜索服务](https://dev.qweather.com/docs/api/geoapi/poi-lookup/)获取。例如 `location=P2951`
* `date`(必选)选择日期，最多可选择未来10天（包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`

### 示例代码

```java
QWeather.getOceanTide(Context context, String location, String date, OnResultOceanTideListener listener);
```

### TideBean属性

| 属性            | 说明     | 示例值                    |
| --------------- | -------- | ---------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)  | 200       |
| getBasic         | 更新信息 | Basic       |
| getRefer         | Refer 数据来源以及数据授权 | Refer  |
| getTideHourlyList | 潮汐小时数据 | List\<TideHourlyBase> |
| getTideTable | 满潮或干潮数据 | List\<TideTableBase> |

**Basic**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00      |
| getFxLink | 当前数据的响应式页面，便于嵌入网站或应用  | https://www.qweather.com |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |


**TideTableBase**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getFxTime      | 满潮或干潮时间                                 | 2017-10-25T04:34+08:00|
| getHeight        | 海水高度，单位：米                                       | 1.23            |
| getType       | 满潮（H）或干潮（L）                              |    H    |

**TideHourlyBase**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getFxTime      | 逐小时预报时间                                 | 2017-10-25T04:34+08:00|
| getHeight        | 海水高度，单位：米                                       | 1.23            |

## 潮流

未来10天全球潮流数据，包括潮流流速和流向。

| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getOceanCurrents| 潮流数据  | CurrentsBean |

### 接口参数说明

* `location`(必选)需要查询的潮流站点，请填写潮流站点的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)，LocationID可通过[POI搜索服务](https://dev.qweather.com/docs/api/geoapi/poi-lookup/)获取。例如 `location=P66981`
* `date`(必选)选择日期，最多可选择未来10天（包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`

### 示例代码

```java
QWeather.getOceanCurrents(Context context, String location, String date, OnResultOceanTideListener listener);
```

### CurrentsBean属性

| 属性            | 说明     | 示例值                    |
| --------------- | -------- | ---------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)  | 200        |
| getBasic         | 更新信息 | Basic       |
| getRefer         | Refer 数据来源以及数据授权 | Refer  |
| getHourlyList | 潮流小时数据 | List\<CurrentsHourlyBase> |
| getTableList | 潮流数据 | List\<CurrentsTableBase> |

**Basic**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00      |
| getFxLink | 当前数据的响应式页面，便于嵌入网站或应用  | https://www.qweather.com |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |


**CurrentsTableBase**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getFxTime      | 潮流最大流速时间                                 | 2017-10-25T04:34+08:00|
| getSpeedMax        | 潮流最大流速，单位：厘米/秒              | 1.23            |
| getDir360       | 潮流360度方向                              |    212    |

**CurrentsHourlyBase**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getFxTime      | 逐小时预报时间                                 | 2017-10-25T04:34+08:00|
| getSpeed        | 潮流流速，单位：厘米/秒              | 1.23            |
| getDir360       | 潮流360度方向                              |    212    |

## 台风列表

台风列表提供全球主要海洋流域最近2年的台风列表。

> 目前仅支持中国沿海地区，即`basin=NP`

| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getStormList| 获取台风列表和ID  | StormListBean |

### 接口参数说明

* `basin`(必选)需要查询的台风所在的流域，例如中国处于西北太平洋，即 `basin=NP`。当前仅支持`NP`。
  * `AL` North Atlantic 北大西洋
  * `EP` Eastern Pacific 东太平洋
  * `NP` NorthWest Pacific 西北太平洋
  * `SP` SouthWestern Pacific 西南太平洋
  * `NI` North Indian 北印度洋
  * `SI` South Indian 南印度洋
* `year`(必选)支持查询本年度和上一年度的台风，例如：`year=2020`, `year=2019`


### 示例代码

```java
QWeather.getStormList(Context context, String year, Basin basin, OnResultTropicalStormListListener listener);
```

### StormListBean属性

| 属性            | 说明     | 示例值                    |
| --------------- | -------- | ---------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)  | 200       |
| getBasic         | 更新信息 | Basic       |
| getRefer         | Refer 数据来源以及数据授权 | Refer  |
| getStormList | 台风数据 | List<StormBean> |


**Basic**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00      |
| getFxLink | 所查询城市的天气预报网页  | https://www.qweather.com |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |


**StormBean**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getId      | 台风ID                               | NP_2101 |
| getName        | 台风名称                                      | 杜鹃           |
| getBasin       | 台风所处流域                              |    NP    |
| getYear       | 台风所处年份                              |    2021    |
| getActive       | 是否为活跃台风<br />`1` 活跃台风 <br /> `0` 停编                              |    0    |

## 台风实况和路径

台风实况和路径提供全球主要海洋流域的台风实时位置、等级、气压、风力、速度以及活跃台风的轨迹路径。

> 目前仅支持中国沿海地区，即`basin=NP`


| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getStormTrack| 台风实况和路径数据  | StormTrackBean |

### 接口参数说明

* `stormid`(必选)需要查询的台风ID，StormID可通过[台风列表](#storm-list)获取。例如 `stormid=NP2018`

### 示例代码

```java
QWeather.getStormTrack(Context context, String stormId, OnResultTropicalStormTrackListener listener);
```

### StormTrackBean属性

| 属性            | 说明     | 示例值                    |
| --------------- | -------- | ---------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)  | 200       |
| getBasic         | 更新信息 | Basic       |
| getRefer         | Refer 数据来源以及数据授权 | Refer  |
| getIsActive         | 是否为活跃台风<br />`1` 活跃台风 <br /> `0` 停编 | 0  |
| getNow | 台风当前信息数据 | StormTrackNowBean |
| getTrackList | 台风路径数据 | List<StormTrackBaseBean> |

**Basic**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00      |
| getFxLink | 所查询城市的天气预报网页  | https://www.qweather.com |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |


**StormTrackNowBean**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getPubTime      | 台风信息发布时间                                 | 2020-10-29T20:00+08:00 |
| getLat        | 台风所处纬度        | 22.5         |
| getLon       | 台风所处经度                           |    148.5    |
| getType       | 台风类型                        |    TS    |
| getPressure  | 台风中心气压                         |  1000 |
| getWindSpeed       | 台风附近最大风速                       |  18    |
| getMoveSpeed       | 台风移动速度                   |   27   |
| getMoveDir       | 台风移动方位                      |    西北    |
| getMove360       | 台风移动方位360度方向  ，可能为空          |    332    |
| getWindRadius30       | 台风7级风圈，**可能为空**          |    StormTrackWindBean    |
| getWindRadius50       | 台风10级风圈，**可能为空**       |    StormTrackWindBean    |
| getWindRadius64       | 台风12级风圈，**可能为空**  |    StormTrackWindBean    |


**StormTrackWindBean**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getNeRadius       | 东北半径                    | 350|
| getSeRadius       | 东南半径               | 260                  |
| getSwRadius       | 西南半径                    | 212                   |
| getNwRadius       | 西北半径                    | 212                   |


**StormTrackBaseBean**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getTime      | 台风信息发布时间                                 | 2020-10-29T20:00+08:00 |
| getLat       | 台风所处纬度        | 22.5         |
| getLon       | 台风所处经度                           |    148.5    |
| getType      | 台风类型                        |    TS    |
| getPressure  | 台风中心气压                         |  1000 |
| getWindSpeed       | 台风附近最大风速                       |  18    |
| getMoveSpeed       | 台风移动速度                   |   27   |
| getMoveDir       | 台风移动方位                      |    西北    |
| getMove360       | 台风移动方位360度方向  ，可能为空          |    332    |
| getWindRadius30       | 台风7级风圈    ，可能为空         |    StormTrackWindBean    |
| getWindRadius50       | 台风10级风圈   ，可能为空       |    StormTrackWindBean    |
| getWindRadius64       | 台风12级风圈，可能为空     |    StormTrackWindBean    |

## 台风预报

台风预报提供全球主要海洋流域的热带低气压（台风）的预报信息，包括台风预测位置、等级、气压、风力、速度

> 如果查询的台风已经结束，则返回的数据为空，建议先通过[台风列表](#storm-list)获取台风的状态

| 接口代码| 接口说明          | 数据类  |
| -------- | ---------------- | ------- |
| getStormForecast| 台风预报数据  | StormForecastBean |

### 接口参数说明

* `stormid`(必选)需要查询的台风ID，StormID可通过[台风列表](#storm-list)获取。例如 `stormid=NP2018`

### 示例代码

```java
QWeather.getStormForecast(Context context, String stormId, OnResultTropicalStormForecastListener listener);
```

### TideBean属性

| 属性            | 说明     | 示例值                    |
| --------------- | -------- | ---------------------- |
| getCode         | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)  | 200       |
| getBasic         | 更新信息 | Basic       |
| getRefer         | Refer 数据来源以及数据授权 | Refer  |
| getForecastList | 台风预报数据 | List<StormForecastBaseBean> |

**Basic**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getUpdateTime | 接口更新时间 | 2017-10-25T04:34+08:00      |
| getFxLink | 所查询城市的天气预报网页  | https://www.qweather.com |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |


**StormForecastBaseBean**

| 属性         | 说明                                                                    | 示例值               |
| ------------ | ----------------------------------------------------- | -------------------- |
| getFxTime      | 台风预报时间                                 | 2020-10-29T20:00+08:00 |
| getLat        | 台风所处纬度        | 22.5         |
| getLon       | 台风所处经度                           |    148.5    |
| getType       | 台风类型                        |    TS    |
| getPressure  | 台风中心气压                         |  1000 |
| getWindSpeed       | 台风附近最大风速                       |  18    |
| getMoveSpeed       | 台风移动速度                   |   27   |
| getMoveDir       | 台风移动方位                      |    西北    |
| getMove360       | 台风移动方位360度方向  ，可能为空          |    332    |

## 日出日落

获取未来60天全球任意地点日出日落时间。

| 接口代码| 接口说明          | 数据类      |
| -------------- | ---------- | ----------- |
| getSun| 日出日落  | SunBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `date`(必选)选择日期，最多可选择未来60天（包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`

### 示例代码

```java
QWeather.getSun(Context context, String location, String date, final OnResultSunListener listener) ;

QWeather.getSun(Context context, String location, Lang lang, String date, final OnResultSunListener listener)                                
```

### SunBean 属性

| 属性                 | 说明                       | 示例值                    |
| -------------------- | -------------------------- | ------------------------- |
| getCode              | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200       |
| getRefer             | Refer 数据来源以及数据授权 | Refer                     |
| getSunrise           | [日出时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空**                   | 2017-10-25T06:01+08:00           |
| getSunset            | [日落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#sunrise-and-sunset)，**在高纬度地区可能为空**                   | 2017-10-25T18:01+08:00           |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | [数据最近更新时间](https://dev.qweather.com/docs/resource/glossary/#update-time)             | 2017-10-25T04:34+08:00   |
| getFxLink     | 当前数据的响应式页面，便于嵌入网站或应用 | https://www.qweather.com/weather/beijing-101010100.html |

## 月升月落和月相

获取未来60天全球城市月升月落和逐小时的月相数据。

> 月相已考虑南北半球的差异，不需要再进行转换

| 接口代码| 接口说明          | 数据类      |
| ------ | ---------- | ----------- |
| getMoon| 太阳和月亮数据  | MoonBean |

### 接口参数说明

* `location`(必选)需要查询地区的[LocationID](https://dev.qweather.com/docs/resource/glossary/#locationid)或以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位），LocationID可通过[GeoAPI](https://dev.qweather.com/docs/api/geoapi/)获取。例如 `location=101010100` 或 `location=116.41,39.92`
* `date`(必选)选择日期，最多可选择未来60天（包含今天）的数据。日期格式为yyyyMMdd，例如 `date=20200531`

### 示例代码

```java
QWeather.getMoon(Context context, String location, String date, final OnResultMoonListener listener) ;

QWeather.getMoon(Context context, String location, Lang lang, String date, final OnResultMoonListener listener)                                
```

### MoonBean 属性

| 属性                 | 说明                       | 示例值                    |
| -------------------- | -------------------------- | ------------------------- |
| getCode              | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200       |
| getRefer             | Refer 数据来源以及数据授权 | Refer                     |
| getMoonrise       | 当天[月升时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空**                   | 2017-10-25T01:34+08:00           |
| getMoonset       | 当天[月落时间](https://dev.qweather.com/docs/resource/sun-moon-info/#moonrise-and-moonset)，**可能为空**                   | 2017-10-25T04:34+08:00           |
| getMoonPhaseBeanList | 月相信息                   | List\<MoonPhaseBean> |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/weather/beijing-101010100.html |


**MoonPhaseBean**

| 属性            | 说明                   | 示例值                 |
| --------------- | ---------------------- | ---------------------- |
| getFxTime       | 月相逐小时预报时间     | 2013-12-31T23:31+08:00 |
| getValue        | 月相数值               | 0.25                   |
| getName         | [月相名称](https://dev.qweather.com/docs/resource/sun-moon-info/#moon-phase)               | 上弦月                 |
| getIllumination | 月亮照明度，百分比数值 | 30                     |
| getIcon | 月相[图标代码](https://dev.qweather.com/docs/resource/icons/)，另请参考[天气图标项目](https://icons.qweather.com/) | 802                     |

## 太阳高度角

任意时间点的全球太阳高度及方位角。

| 接口代码| 接口说明          | 数据类      |
| ------ | ---------- | ----------- |
| getSolarElevationAngle| 太阳高度角数据  | SolarElevationAngleBean |

### 接口参数说明

* `location`(必选)需要查询地区的以英文逗号分隔的[经度,纬度坐标](https://dev.qweather.com/docs/resource/glossary/#coordinate)（十进制，最多支持小数点后两位）。例如 `location=116.41,39.92`
* `date`(必选)查询日期，格式为yyyyMMdd，例如 `date=20170809`
* `time`(必选)查询时间，格式为HHmm，24时制，例如 `time=1230`
* `tz`(必选)查询地区所在时区，例如`tz=0800`或`tz=-0530`
* `alt`(必选)海拔高度，单位为米，例如`alt=43`

### 示例代码

```java
QWeather.getSolarElevationAngle(Context context, String location, String date, String time, String timezone, String alt, final OnResultSolarElevationAngleListener listener)                                
```

### SolarElevationAngleBean属性

| 属性                 | 说明                       | 示例值                    |
| -------------------- | -------------------------- | ------------------------- |
| getCode              | 参考[状态码](https://dev.qweather.com/docs/resource/status-code/)                    | 200       |
| getRefer             | Refer 数据来源以及数据授权 | Refer                     |
| getSolarElevationAngle       | 太阳高度角                   | 70.73  |
| getSolarAzimuthAngle       |  太阳方位角，正北顺时针方向角度   | 205.95      |
| getSolarHour | 太阳时，HHmm格式                 | 1217 |
| getHourAngle | 时角                   | -4.41 |

**Refer**

| 属性           | 说明         | 示例值             |
| -------------- | ------------ | ------------------ |
| getSourcesList | 原始数据来源 | QWeather      |
| getLicenseList | 使用许可     | QWeather Developers License |

**Basic**

| 属性          | 说明                     | 示例值               |
| ------------- | ------------------------ | -------------------- |
| getUpdateTime | 接口更新时间             | 2017-10-25T04:34     |
| getFxLink     | 所查询城市的天气预报网页 | https://www.qweather.com/weather/beijing-101010100.html |
