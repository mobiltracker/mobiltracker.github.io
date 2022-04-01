# Utils

## Introduction

The goal of this documentation is to set a standard for `utils` in our projects.

## Where to find them

`utils` must be located inside a folder called `src/utils`. See an [example](https://github.com/mobiltracker/mobiltracker-partner/tree/master/Mobiltracker.Partner.Web.Core/client/src/utils).

## Common Utils

- [Phone number formatter](#phone-number-formatter)
- [CPF-CNPJ formatter](#cpf-cpnj-formatter)
- [Email Validator](#email-validator)

### Phone number formatter

```Typescript
const formatPhoneNumber = (input: string) => {
  let cleaned = input.replace(/\D/g, "");
  let match = cleaned.match(/^(\d{2})(\d{5})(\d{4})$/);

  if (match) {
    return "(" + match[1] + ") " + match[2] + "-" + match[3];
  } else {
    return cleaned;
  }
};
```

### CPF-CPNJ formatter

```Typescript
const formatCpfCnpj = (input: string) => {
  let cleaned = input.replace(/\D/g, "");
  let cpfMatch = cleaned.match(/^(\d{3})(\d{3})(\d{3})(\d{2})$/);
  let cnpjMatch = cleaned.match(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/);

  if (cpfMatch) {
    return `${cpfMatch[1]}.${cpfMatch[2]}.${cpfMatch[3]}-${cpfMatch[4]}`;
  } else if (cnpjMatch) {
    return `${cnpjMatch[1]}.${cnpjMatch[2]}.${cnpjMatch[3]}/${cnpjMatch[4]}-${cnpjMatch[5]}`;
  } else {
    return cleaned;
  }
};
```

### Email Validator

```Typescript
function validateEmail(value: string) {
  return /^[a-zA-Z0-9.+_-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)+$/.test(value);
};
```