---
title: IDL
author: vwkd
index: 3
tags:
  - languages
  - graphql
---

<!-- todo: finish -->

- defines API, e.g. supported operations, fields, directives, etc.
- ??? uses type system
- ??? beware: don't confuse with query language, e.g. object fields and object field types
syntax of schema vs. syntax of request

??? checks if request is valid, e.g. query field that doesn't exist, type of variable, etc.



## Type System

??
- convention to write names lowerCamelCase (e.g. field, argument, etc.) and types UpperCamelCase

```plaintext
TypeSystemDefinition:
   SchemaDefinition TypeDefinition*+ DirectiveDefinition*+
```

- beware: differs from spec to allow multiple `TypeDefinition`s or `DirectiveDefinition`s in a `TypeSystemDefinition` instead of using multiple `TypeSystemDefinition`s in multiple `Definition`s, also only single `SchemaDefinition`, see [#788](https://github.com/graphql/graphql-spec/issues/788#issuecomment-721298885) ❗️

### Extension

- extension of type system, e.g. by local service
- can use to add new fields, interfaces, directives, etc.

```plaintext
TypeSystemExtension:
    SchemaExtension*+ TypeExtension*+

SchemaExtension: 
    "extend schema" Directives* { OperationTypeDefinition+ }
    "extend schema" Directives

TypeExtension:
    ScalarTypeExtension
    ObjectTypeExtension
    InterfaceTypeExtension
    UnionTypeExtension
    EnumTypeExtension
    InputObjectTypeExtension
```

- beware: differs from spec to allow multiple `SchemaExtension`s or `TypeExtension`s in a `TypeSystemExtension` instead of using multiple `TypeSystemExtension`s, see [#788](https://github.com/graphql/graphql-spec/issues/788#issuecomment-721298885) ❗️



## Schema

- definition of type system
- ??? uses types and directives

```plaintext
SchemaDefinition:
    Description* "schema" Directives* { OperationTypeDefinition+ }

OperationTypeDefinition
    QueryOperationTypeDefinition MutationOperationTypeDefinition* SubscriptionOperationTypeDefinition*

QueryOperationTypeDefinition:
    "query" : NamedType

MutationOperationTypeDefinition:
    "mutation" : NamedType

SubscriptionOperationTypeDefinition:
    "subscription" : NamedType
```

- beware: in spec calls "root operation type", here leave out "root" since not needed ❗️
- beware: differ from spec to allow at most one `OperationTypeDefinition` for each `OperationType` ❗️
- beware: can't have multiple query operations, structures independent data hierarchies as fields of query operation type ❗️
- operation type `NamedType` must be an object type
- beware: operation type is nothing special, objects all the way down ❗️
- must define at least one query operation, other operations are optional
- different operation types must be different types, e.g. query operation type can't be same as mutation operation type
??? what about subscription, can have same type as query operation ???

```graphql
schema {
  query: MyQuery
  mutation: MyMutation
}

type MyQuery {
  myField: String
}

type MyMutation {
  setMyField(to: String): String
}
```

- shorthand: for schema with names of operation types identical to name of operation, can leave out schema

```graphql
# `schema { query: Query }` implicitly defined

type Query {
  myField: String
}
```

- by convention, names operation type identical to operation, uses shorthand schema definition
- beware: shorthand silently ignores other types, e.g. due to misspelling ❗️

```graphql
# `schema { query: Query }` implicitly defined

type Query {
  myField: String
}

# Silently ignored, since doesn't match "Mutation" (or "Subscription")
type Mutaion {
  setMyField(to: String!): String
}
```

- beware: shorthand disallows use name "Query", "Mutation" or "Subscription" for other type than operation type, although shouldn't do that anyways to avoid confusion ❗️

```graphql
# `schema { query: Query, subscription: Subscription }` implicitly defined

type Query {
  myField: Subscription
}

# Becomes operation type unintendedly
type Subscription {
  nyField: String
}
```



## Output Type

- type of field, specifies values in response
- similar to TypeScript, e.g. object types, interface types, union types, etc.
- can use in type system where references `NamedType`
- name of type, field, argument must be unique, must not start with two underscores since reserved for introspection
- fields of primitive type are leaves in response tree, fields of composite type are intermediate nodes

### Named types

??

```plaintext
TypeDefinition:
    ScalarTypeDefinition
    ObjectTypeDefinition
    InterfaceTypeDefinition
    UnionTypeDefinition
    EnumTypeDefinition
    InputObjectTypeDefinition
```

- named type is nullable (includes `null` value) and singular, can use wrapping type to make non-nullable and/or list
- beware: type being nullable by default is opposite of most type systems like TypeScript, e.g. optional type `T?` for type `T`
- non-nullable type guarantees field being present in response and not being `null` value ❗️
- beware: nullable type allows both omission or `null` value, non-nullable type neither allows omission nor `null` value, can't have only one ❗️
- beware: `null` value and omission both describe lack of value, but separate things ❗️
- beware: doesn't use `type` keyword except for object type even though are all types ⚠️

#### Primitive types

- scalar

```plaintext
ScalarTypeDefinition:
    Description* "scalar" Name Directives*
```

- built-in scalars in any schema engine: `Int`, `Float`, `String`, `Boolean`, `ID`
- beware: build own scalar types that needs, e.g. `Date` string, `URL` string ❗️
- enum

```plaintext
EnumTypeDefinition:
    Description* "enum" Name Directives* EnumValuesDefinition*

EnumValuesDefinition:
    { EnumValueDefinition+ }

EnumValueDefinition:
    Description* EnumValue Directives*
```

- by convention, name in `EnumValue` is UPPERCASE

#### Composite types

- object

```plaintext
ObjectTypeDefinition:
    Description* "type" Name ImplementsInterfaces* Directives* FieldsDefinition*

ImplementsInterfaces:
    ImplementsInterfaces & NamedType
    "implements" &* NamedType

FieldsDefinition:
    { FieldDefinition+ }

FieldDefinition:
    Description* Name ArgumentsDefinition* : Type Directives*

ArgumentsDefinition:
    ( InputValueDefinition+ )

InputValueDefinition:
    Description* Name : Type DefaultValue* Directives*
```

- `NamedType` in `ImplementsInterfaces` must be name of an interface, see Abstract types
- `Type` in `InputValueDefinition` must be an input type, see Input Type


#### Abstract types

- interface: abstract object type

```plaintext
InterfaceTypeDefinition:
    Description* "interface" Name ImplementsInterfaces* Directives* FieldsDefinition*
```

- object type that implements interface must implement all its fields
- interface can't implement other interface that creates a cyclic reference, e.g. itself
- can use interface as type for field, field can return any object type that implements that interface
- beware: if uses an interface as type for field, can only query fields that are on interface, i.e. guaranteed to be implemented by return type, or use fragment with type condition ❗️

```graphql
# part of schema
interface NamedEntity {
  name: String
}

type Person implements NamedEntity {
  name: String
  age: Int
}

type Contact {
  entity: NamedEntity
  phoneNumber: String
  address: String
}
```

```graphql
# operation
{
  entity {
    name
  }
  phoneNumber
}
```

```graphql
{
  entity {
    name
    ... on Person {
      age
    }
  },
  phoneNumber
}
```

- union

```plaintext
UnionTypeDefinition:
    Description* "union" Name Directives* UnionMemberTypes*
UnionMemberTypes:
    UnionMemberTypes | NamedType
    = |* NamedType
```

- beware: `NamedType` must be an object type and not abstract types, e.g. interface, union, etc., currently doesn't allow union of primitive types, see [#215](https://github.com/graphql/graphql-spec/issues/215) ⚠️
- beware: doesn't use `type` keyword even though is a type ⚠️
- can use union as type for field, field can return any object type in union
- beware: if uses a union as type for field, can query no fields without using fragment with type condition, even if all members of union have that field, differs from TypeScript where can access a property that is on all members of a union of object types ⚠️

### Wrapping types

- wrap an existing type

#### List type

- collection of values of given type
- syntax: `[ T ]` for type `T`
- field of list type is pluralised by convention

```graphql
# request
{
  users(first: 5) {
    name
  }
}
```

```json
// response
{
    "users": [
        {
            "name": "Peter"
        },
        // ...
        {
            "name": "Sarah"
        }
    ]
}
```

```graphql
# schema
type Query {
    users(first: Int!): [User]
}

type User {
    name: String
}
```

- beware: every field of list type should use pagination if list can change, see Pagination ❗️

#### Non-nullable type

- given type without `null` value
- syntax: `T!` for type `T`



## Input Type

- type of argument (or variable), specifies values in request
- i.e. scalar, enum, (input) object
- name of type, field must be unique, must not start with two underscores since reserved for introspection

### Named types

- as in Output Type, see Output Type
- non-nullable type makes argument (or variable) required and not `null` value, e.g. input for mutation operation
- beware: `null` value and omission both describe lack of value, but separate things ❗️
- beware: doesn't use `type` keyword even though are all types ⚠️

#### Primitive types

- as in Output Type, see Output Type

#### Composite types

- input object

```plaintext
InputObjectTypeDefinition:
    Description* "input" Name Directives* InputFieldsDefinition*

InputFieldsDefinition:
    { InputValueDefinition+ }
```

- beware: don't confuse with (output) object type, doesn't allow field arguments or references to abstract types ❗️
- beware: field corresponds to property in input object instead of in output object ❗️

### Wrapping types

- as in Output Type, see Output Type



## Directive

```plaintext
DirectiveDefinition
    Description* "directive" @ Name ArgumentsDefinition* "on" DirectiveLocations

DirectiveLocations:
    |* DirectiveLocation
    DirectiveLocations | DirectiveLocation

DirectiveLocation:
    ExecutableDirectiveLocation
    TypeSystemDirectiveLocation

ExecutableDirectiveLocation:
    "QUERY" OR "MUTATION" OR "SUBSCRIPTION" OR "FIELD" OR "FRAGMENT_DEFINITION" OR "FRAGMENT_SPREAD" OR "INLINE_FRAGMENT" OR "VARIABLE_DEFINITION"

TypeSystemDirectiveLocation:
    "SCHEMA" OR "SCALAR" OR "OBJECT" OR "FIELD_DEFINITION" OR "ARGUMENT_DEFINITION" OR "INTERFACE" OR "UNION" OR "ENUM" OR "ENUM_VALUE" OR "INPUT_OBJECT" OR "INPUT_FIELD_DEFINITION"
```

- name of directive must be unique, not start with two underscores since reserved for introspection
- directive can't use other directive in argument that creates a cyclic reference, e.g. itself
- built-in directives in any schema engine:
    - `@deprecated(reason: String)` on field or enum: marks field or enum value as deprecated in schema
- beware: currently spec doesn't include `@deprecated` on argument, see [#525](https://github.com/graphql/graphql-spec/pull/525)



## Description

- strings provided in documentation alongside definition
- directly in code, self-documenting, no need to keep documentation in sync 🎉
- can add immediately before schema, type, field in object, enum value in enum, argument, directive
- think of JSDoc comments in TypeScript
- can use markdown

```plaintext
Description
    StringValue
```



## Resources

- [GraphQL - Specification 11/2020](http://spec.graphql.org/draft/)