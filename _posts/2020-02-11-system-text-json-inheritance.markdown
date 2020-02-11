---
layout: post
title:  "Serializing derived classes using System.Text.Json.JsonSerializer"
date:   2020-02-11 22:00:00 +0000
categories: json system.text.json
---
By default, ASP.NET Core uses a new JSON serializer: [System.Text.Json.JsonSerializer](https://docs.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializer). This has slightly different behaviour to Newtonsoft's [Json.NET](https://www.newtonsoft.com/json) when serializing derived types.

### Example Code
{% highlight csharp %}
public class Program
{
    public class BaseClass
    {
        public string BaseClassProperty { get; set; }
    }

    public class DerivedClass
    {
        public class DerivedClassProperty { get; set; }
    }

    public static void Main(string[] args)
    {
        BaseClass toSerialize = new DerivedClass()
            {
                BaseClassProperty = "One",
                DerivedClassProperty = "Two"
            };
        var serialized = JsonSerializer.Serialize(toSerialize);
        
        Console.WriteLine(serialized);
    }
}
{% endhighlight %}
This will output `{ "baseClassProperty": "One" }`; note the absence of the `derivedClassProperty` field. Using Newtonsoft's Json.NET library to serialize the same object will include both properties.

### Workaround
To work around it, you can make a polymorphic JsonConverter. Sample code for this is in a reply to the GitHub issue, located [here](https://github.com/dotnet/runtime/issues/29937#issuecomment-548517428).

I added an extra check to handle serializing null values, turning the class into:
{% highlight csharp %}
public class PolymorphicJsonConverter<T> : JsonConverter<T>
{
    public override T
        Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        throw new NotImplementedException();
    }
    
    public override void
        Write(Utf8JsonWriter writer, T value, JsonSerializerOptions options)
    {
        JsonSerializer.Serialize(writer, value, value?.GetType() ?? typeof(T), options);
    }
}
{% endhighlight %}
