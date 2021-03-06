# ELCodable 

[![Version](https://img.shields.io/badge/version-v3.1.0-blue.svg)](https://github.com/Electrode-iOS/ELCodable/releases/latest)
[![Build Status](https://travis-ci.org/Electrode-iOS/ELCodable.svg?branch=master)](https://travis-ci.org/Electrode-iOS/ELCodable)
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

ELCodable, a data model decoding/encoding framework for Swift. Inspired by [Anviking's Decodable](https://github.com/Anviking/Decodable). ELCodable provides a mechanism for decoding and encoding JSON to and from Swift models.

* Swift optionals to determine required fields from optional fields.
* An easy to use JSON wrapper.
* Encoding to JSON.
* Decoding from JSON.
* Type conversion, both in model types as well as common forms of JSON, such as NSData, Dictionaries, Arrays, etc.
* Data validation.

## Requirements

ELCodable requires Swift 3 and Xcode 8.3.

## Installation

### Carthage

Install with [Carthage](https://github.com/Carthage/Carthage) by adding the framework to your project's [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile).

```
github "Electrode-iOS/ELCodable" ~> 3.1.0
```

### Manual

Install manually by adding `ELCodable.xcodeproj` to your project and configuring your target to link `ELCodable.framework`.

## Usage

### Defining & using your model

```Swift
struct MyModel {
    let myString: String
    let myNumber: UInt
}
```

Once you've defined your model, now extend it such that it works with the decoder.

```Swift
extension MyModel: Decodable {
    static func decode(json: JSON?) throws -> MyModel {
        return try MyModel(
            myString: json ==> "myString",
            myNumber: json ==> "myNumber"
        )
    }
}
```

The above will allow your model to be decoded.  As for triggering decoding, you've got a few options...

The simplest way:
```Swift
let myModel = try? MyModel.decode(json)
```

At this point, myModel will either have a value, or be nil.  If you'd like more information on what caused a failure, you can do this:

```Swift
do {
    // decode the json
    let myModel = try MyModel.decode(json)
    // do something with the model
    doSomething(myModel)
} catch DecodeError.NotFound(let key) {
    print("MyModel couldn't be decoded because \(key) couldn't be found.")
} catch let error {
    // catch all for any errors that may happen.
}
```

Now that we've decoded and done something with our model.  Lets look at how encoding would work.

### Encoding

```Swift
extension MyModel: Encodable {
    func encode() throws -> JSON {
        return try encodeToJSON(
            "myString" <== myString,
            "myNumber" <== myNumber
        )
    }
}
```

Now that you've done this, you can send it to disk wherever else it might need to go.

```Swift
let json = try? myModel.encode()
if let json = json {
    json.data().writeToFile(path)
}
```

### Validation

Model decode validation is as easy as adding a call to validateModel(), and writing a validateDecode() function.  Validating Encoding works more or less the same way.

```Swift
extension MyModel: Decodable {
    static func decode(json: JSON?) throws -> MyModel {
        return try MyModel(
            myString: json ==> "myString",
            myNumber: json ==> "myNumber"
        ).validateModel() // triggers validation to occur
    }
    
    func validateDecode() throws -> MyModel {
        if myNumber != 3 {
            throw DecodeError.ValidationFailed
        } else {
            // myNumber is 3, our model is valid.
            return self
        }
    }
}
```
### What about sub-models?

Take this example model:

```Swift
struct TestModel {
    let aString: String
    let aModelArray: [SubModel]
}
```

As long as SubModel implements Decodable and/or Encodable, it'll "just work".  Validation is left up to the implementor, but it's done the same way regardless of sub-models.

### What about fields that aren't necessarily required?

Simply mark them as optionals in your model, like this:

```Swift
struct TestModel {
    // required
    let aString: String
    // not required
    let aModelArray: [SubModel]?
}
```

### What types can I use in my models?

Model types will typically be Swift types.  Things like String, UInt, structs, Dictionary, etc.  Because of this, it's not suitable for use with Objective-C directly.  It can be done though with a little wrapper. See Decimal.swift, which makes NSDecimalNumber a first class type in Swift.

You may come across other types you'd like to have in your models besides the ones supplied in ELCodable.  It's pretty easy to do this, you'd just need to make the type conform to Decodeable and/or Encodable.  See below:

```Swift
extension Bool: Decodable {
    public static func decode(json: JSON?) throws -> Bool {
        if let value = json?.bool {
            return value
        }
        throw DecodeError.Undecodable
    }
}

extension Decimal: Decodable {
    public static func decode(json: JSON?) throws -> Decimal {
        if let value = json?.decimal {
            return Decimal(value)
        }
        throw DecodeError.Undecodable
    }
}
```

### Tell me more about this Decimal type

The Decimal type allows NSDecimalNumber to work and function exactly as a Double or Float would, while preserving the precision within.  There's not much to it really, however since numbers tend to be compared to various things, it implements the Equatable protocol and has various operator overloads on ==, <=, >= etc.
