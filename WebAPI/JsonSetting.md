### Json  ""转换为null的问题
```
/// <summary>
    /// 解决ConvertEmptyStringToNull默认为true时 ""转换为null的问题
    /// </summary>
    public class MyDataAnnotationsModelMetadataProvider: DataAnnotationsModelMetadataProvider
    {
        protected override ModelMetadata CreateMetadata(IEnumerable<System.Attribute> attributes, Type containerType, Func<object> modelAccessor, Type modelType, string propertyName)
        {
            var md = base.CreateMetadata(attributes, containerType, modelAccessor, modelType, propertyName);

            DataTypeAttribute dataTypeAttribute = attributes.OfType<DataTypeAttribute>().FirstOrDefault();
            DisplayFormatAttribute displayFormatAttribute = attributes.OfType<DisplayFormatAttribute>().FirstOrDefault();
            if (displayFormatAttribute == null && dataTypeAttribute != null)
            {
                displayFormatAttribute = dataTypeAttribute.DisplayFormat;
            }
            if (displayFormatAttribute == null)
            {
                md.ConvertEmptyStringToNull = false;
            }

            return md;
        }
    }
 ```

### Json 时间格式

```
 // Create Json.Net formatter serializing DateTime using the ISO 8601 format
            config.Formatters.JsonFormatter.SerializerSettings.Converters.Add(
                new IsoDateTimeConverter() { DateTimeFormat = "yyyy-MM-dd HH:mm:ss" });

```


## Reference
https://www.hanselman.com/blog/OnTheNightmareThatIsJSONDatesPlusJSONNETAndASPNETWebAPI.aspx   
