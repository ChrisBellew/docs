---
slug: customize-json-responses
title: Customize JSON Responses
---

The JSON Responses for all ServiceStack Services can be configured Globally, individually per-Service or customized per-request by the client using the `?jsconfig` QueryString modifier.

## Global Default JSON Configuration

ServiceStack uses the [ServiceStack.Text Serializers](https://github.com/ServiceStack/ServiceStack.Text) for its built-in JSON/JSV and CSV serialization. The serialization can be customized globally by configuring the `JsConfig` or type-specific `JsConfig<T>` static classes with your preferred defaults, e.g:

```csharp
public override void Configure(Container container)
{
    JsConfig.EmitLowercaseUnderscoreNames = true; 
    JsConfig.ExcludeDefaultValues = true;

    JsConfig<Guid>.SerializeFn = guid => guid.ToString("D");
    JsConfig<TimeSpan>.SerializeFn = time => 
        (time.Ticks < 0 ? "-" : "") + time.ToString("hh':'mm':'ss'.'fffffff");
}
```

## Customize JSON Responses in Service

The Global Defaults can be overridden on a adhoc basis by returning your Response DTO in a custom `HttpResult` configured with a custom JS Config Scope, e.g:

```csharp
return new HttpResult(responseDto) {
    ResultScope = () => JsConfig.With(
        emitLowercaseUnderscoreNames:true, excludeDefaultValues:true)
};
```

## Customize JSON Responses from Client

The JSON and JSV Responses for all Services (inc. [Auto Query](/autoquery) Services) can also be further customized with the 
new `?jsconfig` QueryString param which lets your Service consumers customize the returned JSON Response to 
their preference. This works similar to having wrapped your Service response in a `HttpResult` with a Custom 
`ResultScope` in the Service implementation to enable non-default customization of a Services response, e.g:

    /service?jsconfig=EmitLowercaseUnderscoreNames,ExcludeDefaultValues
    
Works similarly to:

```csharp
return new HttpResult(new { TheKey = "value", Foo=0 }) {
    ResultScope = () => JsConfig.With(
        emitLowercaseUnderscoreNames:true, excludeDefaultValues:true)
};
```

Which results in **lowercase_underscore** key names with any properties with **default values removed**:

    {"the_key":"value"}

It also supports cascading server and client ResultScopes, with the client `?jsconfig` taking precedence.

Nearly all `JsConfig` scope options are supported other than delegates and complex type configuration properties.

### Camel Humps Notation

JsConfig also supports Camel Humps notation letting you target a configuration by just using the 
**Uppercase Letters** in the property name which is also case-insensitive so an equivalent shorter version 
of the above config can be:

    ?jsconfig=ELUN,edv
    
Camel Humps also works with Enum Values so both these two configurations are the same:

    ?jsconfig=DateHandler:UnixTime
    ?jsconfig=dh:ut

### Custom JSON Live Example

[AutoQuery Viewer](https://github.com/ServiceStack/Admin) makes use of this feature in order to return human readable dates using the new 
`ISO8601DateOnly` DateHandler Enum Value as well as appending `ExcludeDefaultValues` when specifying custom 
fields so that any unpopulated value type properties with default values are excluded from the JSON Response. 
Here's a live example of this comparing the default Response with the customized JSON Response:

 - [github.servicestack.net/repos.json?fields=Name,Homepage,Language,Updated_At](http://github.servicestack.net/repos.json?fields=Name,Homepage,Language,Updated_At)
 - [github.servicestack.net/repos.json?fields=Name,Homepage,Language,Updated_At&jsconfig=edv,dh:iso8601do](http://github.servicestack.net/repos.json?fields=Name,Homepage,Language,Updated_At&jsconfig=edv,dh:iso8601do)

### Custom JSON Settings

The presence of a **bool** configuration property will be set to `true` unless they have a `false` or `0` 
value in which case they will be set to `false`, e.g:

    ?jsconfig=ExcludeDefaultValues:false

For a quick reference the following **bool** customizations are supported:

<table>
    <thead>
        <tr><th>Name</th><th>Alias</th></tr>
    </thead>
    <tr><td>EmitCamelCaseNames</td><td>eccn</td></tr>
    <tr><td>EmitLowercaseUnderscoreNames</td><td>elun</td></tr>
    <tr><td>IncludeNullValues</td><td>inv</td></tr>
    <tr><td>IncludeNullValuesInDictionaries</td><td>invid</td></tr>
    <tr><td>IncludeDefaultEnums</td><td>ide</td></tr>
    <tr><td>IncludePublicFields</td><td>ipf</td></tr>
    <tr><td>IncludeTypeInfo</td><td>iti</td></tr>
    <tr><td>ExcludeTypeInfo</td><td>eti</td></tr>
    <tr><td>ConvertObjectTypesIntoStringDictionary</td><td>cotisd</td></tr>
    <tr><td>TreatEnumAsInteger</td><td>teai</td></tr>
    <tr><td>TryToParsePrimitiveTypeValues</td><td>ttpptv</td></tr>
    <tr><td>TryToParseNumericType</td><td>ttpnt</td></tr>
    <tr><td>ThrowOnDeserializationError</td><td>tode</td></tr>
    <tr><td>EscapeUnicode</td><td>eu</td></tr>
    <tr><td>PreferInterfaces</td><td>pi</td></tr>
    <tr><td>SkipDateTimeConversion</td><td>sdtc</td></tr>
    <tr><td>AlwaysUseUtc</td><td>auu</td></tr>
    <tr><td>AssumeUtc</td><td>au</td></tr>
    <tr><td>AppendUtcOffset</td><td>auo</td></tr>
</table>

### DateHandler (dh)

<table>
    <tr><td>TimestampOffset</td><td>to</td></tr>
    <tr><td>DCJSCompatible</td><td>dcjsc</td></tr>
    <tr><td>ISO8601</td><td>iso8601</td></tr>
    <tr><td>ISO8601DateOnly</td><td>iso8601do</td></tr>
    <tr><td>ISO8601DateTime</td><td>iso8601dt</td></tr>
    <tr><td>RFC1123</td><td>rfc1123</td></tr>
    <tr><td>UnixTime</td><td>ut</td></tr>
    <tr><td>UnixTimeMs</td><td>utm</td></tr>
</table>

### TimeSpanHandler (tsh)

<table>
    <tr><td>DurationFormat</td><td>df</td></tr>
    <tr><td>StandardFormat</td><td>sf</td></tr>
</table>

### PropertyConvention (pc)

<table>
    <tr><td>Strict</td><td>s</td></tr>
    <tr><td>Lenient</td><td>l</td></tr>
</table>

#### Create Custom Scopes using String config

You can also create a scope from a string manually using `JsConfig.CreateScope()`, e.g:

```csharp
using (JsConfig.CreateScope("EmitLowercaseUnderscoreNames,ExcludeDefaultValues,dh:ut")) 
{
    var json = dto.ToJson();
}
```

If you don't wish for consumers to be able to customize JSON responses this feature can be disabled with 
`Config.AllowJsConfig=false`.