# Conditional Types - Typescript

## Problem

Given a type like the one below:

```Typescript
type Parent = {
  option: "A" | "B";
  children: { subOption: { a: true } | { b: true } };
};
```

It's not possible to guarantee that `children` will have the correct `subOption`.

```Typescript

const foobar: Parent = {
  // this is ok
  option: "A",
  children: { subOption: { a: true } },
};

const foobar2: Parent = {
  // this is wrong, but the compiler cannot ensure the correct type
  option: "A",
  children: { subOption: { b: true } },
};
```

## How to fix this

To guarantee that `children` will have the correct type use [conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html). So, the compiler is able to infer the correct type.

```Typescript
type Parent<Option extends "A" | "B"> = {
  option: Option;
  children: {
    subOption: Option extends "A"
      ? { a: true }
      : Option extends "B"
      ? { b: true }
      : never;
  };
};

const foobar: Parent<"A"> = {
  option: "A",
  children: { subOption: { a: true } },
};

const foobar2: Parent<"A"> = {
  option: "A",
  // @ts-expect-error
  children: { subOption: { b: true } },
```

## Example

Using conditional type we can set the properties of a column by the table's responsive type. Like this:

```Typescript
type ResponsiveType = "shorten" | "movable";

type ResponsiveTypePropsForShorten = { visible: boolean };
type ResponsiveTypePropsForMovable = { move: boolean };

type ResponsiveTypeProps<R extends ResponsiveType> = R extends "shorten"
  ? ResponsiveTypePropsForShorten
  : R extends "movable"
  ? ResponsiveTypePropsForMovable
  : never;

type Table<R extends ResponsiveType> = {
  responsiveType: R;
  columns: Column<R>[];
};

type Column<R extends ResponsiveType> = {
  responsiveTypeProps: ResponsiveTypeProps<R>;
};

const table1: Table<"shorten"> = {
  responsiveType: "shorten",
  columns: [{ responsiveTypeProps: { visible: true } }],
};

const table2: Table<"movable"> = {
  responsiveType: "movable",
  columns: [{ responsiveTypeProps: { move: false } }],
};

```

If you want to see the full version of the example above [click here](https://github.com/mobiltracker/vanescola-portal-admin/blob/f0b41b27f28a44cb5236cd04127d590102577472/src/components/Table/Table.tsx)
