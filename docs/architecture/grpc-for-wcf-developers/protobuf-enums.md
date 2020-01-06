---
title: Protobuf enumerations - gRPC for WCF Developers
description: Learn how to declare and use enumerations in Protobuf.
ms.date: 09/09/2019
---

# Protobuf enumerations

Protobuf supports enumeration types, as seen in the previous section where an enum was used to determine the type of a `oneof` field. You can define your own enumeration types and Protobuf will compile them to C# enum types. Since Protobuf can be used with different languages, the naming conventions for enumerations are different from the C# conventions. However, the code generator is clever and converts the names to the traditional C# case. If the Pascal-case equivalent of the field name starts with the enumeration name, then it's removed.

For example, in this Protobuf enumeration the fields are prefixed with `ACCOUNT_STATUS`, which is equivalent to the Pascal case enum name: `AccountStatus`.

```protobuf
enum AccountStatus {
  ACCOUNT_STATUS_UNKNOWN = 0;
  ACCOUNT_STATUS_PENDING = 1;
  ACCOUNT_STATUS_ACTIVE = 2;
  ACCOUNT_STATUS_SUSPENDED = 3;
  ACCOUNT_STATUS_CLOSED = 4;
}
```

So, the generator creates a C# enum equivalent to the following code:

```csharp
public enum AccountStatus
{
    Unknown = 0,
    Pending = 1,
    Active = 2,
    Suspended = 3,
    Closed = 4
}
```

Protobuf enumeration definitions **must** have a zero constant as their first field. As in C#, you can declare multiple fields with the same value, but you must explicitly enable this option using the `allow_alias` option in the enum:

```protobuf
enum AccountStatus {
  option allow_alias = true;
  ACCOUNT_STATUS_UNKNOWN = 0;
  ACCOUNT_STATUS_PENDING = 1;
  ACCOUNT_STATUS_ACTIVE = 2;
  ACCOUNT_STATUS_SUSPENDED = 3;
  ACCOUNT_STATUS_CLOSED = 4;
  ACCOUNT_STATUS_CANCELLED = 4;
}
```

You can declare enumerations at the top level in a `.proto` file, or nested within a message definition. Nested enumerations—like nested messages—will be declared within the `.Types` static class in the generated message class.

There's no way to apply the [[Flags]](xref:System.FlagsAttribute) attribute to a Protobuf-generated enum, and Protobuf doesn't understand bitwise enum combinations. Take a look at the following example:

```protobuf
enum Region {
  REGION_NONE = 0;
  REGION_NORTH_AMERICA = 1;
  REGION_SOUTH_AMERICA = 2;
  REGION_EMEA = 4;
  REGION_APAC = 8;
}

message Product {
  Region available_in = 1;
}
```

If you set `product.AvailableIn` to `Region.NorthAmerica | Region.SouthAmerica`, it's serialized as the integer value `3`. When a client or server tries to deserialize the value, it won't find a match in the enum definition for `3` and the result will be `Region.None`.

The best way to work with multiple enum values in Protobuf is to use a `repeated` field of the enum type.

>[!div class="step-by-step"]
>[Previous](protobuf-any-oneof.md)
>[Next](protobuf-maps.md)
