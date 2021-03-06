# Thing Data Models in JSON-LD

This is a description of how Thing data models can be defined in JSON-LD and has been implemented by a [NodeJS based server](https://github.com/w3c/web-of-things-framework). See also [Slides describing work on some open source web of things servers](https://www.w3.org/WoT/IG/wiki/images/a/aa/Wot-dsr-demo.pdf).

The syntax assumes a JSON-LD context that maps short names to RDF URLs, e.g. to the RDF core datatypes that RDF imports from XML Schema. The default JSON-LD context is assumed, e.g. on the basis of the Media Type used for the model.  Thing descriptions should use @context to reference domain semantics. This will be ignored on resource constrained devices, but on more powerful platforms, the domain semantics can be used as a basis for selecting matching services, e.g. during service composition.

The following covers an example of an electric light. The model is protocol independent, provides an on/off switch, settable colour temperature, access to the rgb values, and a transition action that over a given number of seconds, changes the brightness and colour temperature to new values. The model allows for notification of changes, but the API for that would be platform dependent as I don’t believe it should be formally treated as an event in the model.

To allow the example to fully conform to the [JSON-LD Recommendation](http://www.w3.org/TR/json-ld/) concrete examples need to be provided for the default context and the domain model. The latter needs to provide URL bindings for the domain terms e.g. “on”, “color_temp” and “dim”. More generally, it could link to ontologies that describe semantic relationships, e.g. that an LED-lamp is a kind of electrically operated light source. The default context would define bindings for “name”, “properties”, “type”, “writeable”, “boolean”, "float" and so forth. You can see examples for how to do this in [section 5.1](http://www.w3.org/TR/json-ld/#the-context) of the JSON-LD specification.

Here is a data model for the light for discussion purposes:

```
{
            "name" : "MyLED",
            "@context" : "http://example.com/domain-semantics",
            "properties": {
                        "on" : {
                                   "type" : "boolean",
                                   "writeable": true
                        },
                        "color_temp": {
                                   "type" : "uint16",
                                   "writeable" : true
                        },
                        "red" : "uint8",
                        "green" : "uint8",
                        "blue" : "uint8"
            },
            "actions" : {
                        "transition" : {
                                    "input" : {
                                                "brightness" : "float",
                                                "color_temp" : "uint16",
                                                "time" : "float"
                                    },
                                    "output" : null
                        }
            }
}
```

This has a name "MyLED" for the specific light.  It has two properties that scripts can write to: "on" is a boolean and "color_temp" is the color temperature in Kelvin. Scripts can read but not write the corresponding red, green and blue values as unsigned 8 bit numbers.

There is one action "transition" which changes the brightness and colour temperature over a given number of seconds. This action takes three named parameters. The target "brightness" is passed as a 32 bit IEEE floating point number in the range 0.0 to 1.0. The target colour temperature is passed as a 16 bit unsigned integer in degrees Kelvin. The "time" for the change to complete is specified in seconds as a 32 bit IEEE floating point number. This action returns no data.

This example has no events.

## Notes

* I have kept things simple for now and would expect this to evolve incrementally as we tackle more complex use cases.
* When a property requires more than the name of a data type, you use a JSON object whose properties provide further information, e.g. {“type”:”float”, “min”:0.0, “max”:1.0}.
* When a property value is given as the name of a data type rather than as a JSON object, there is an implicit "type" relationship between the property and its type. This is syntactic sugar for a JSON object with a single field named "type" whose value is the name of the type.
* For nested properties, you define the property value as a JSON object, one of whose fields is "properties". Its value is a JSON object whose fields are the nested property names.
* Further work is needed to specify metadata on whether a property value is static, or if not, how long its value can be relied on. 
* The server metadata is held separately and describes which protocols, data formats and encodings etc. that the server supports. This is needed to determine which protocols etc. to use to connect to a given server.

## Editing

Manual editing is error prone, the following links can help to identify typo's

* [JSON Lint](http://jsonlint.com) which can be used to validate JSON
* [JSON-LD Playground](http://json-ld.org/playground/) which can be used to validate JSON-LD

p.s. the above model is valid JSON but not yet valid JSON-LD since the example domain model can't be dereferenced. I plan to fix that soon as I get the time.
