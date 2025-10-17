# id-doc-validator

A validator for different types of personal, entity and VAT IDs for multiple countries.

## 🚀 Quick Start Guide

### Installation

Install the package using npm or yarn:

```bash
npm install id-doc-validator
# or
yarn add id-doc-validator
```

### Basic Usage

```javascript
const {
  isValidIdDoc,
  isValidVat,
  isValidViesVat,
  supportedCountriesIdDoc,
  supportedCountriesVat,
  supportedIdDocsByCountry
} = require('id-doc-validator');
```

## 📖 Usage Examples

### Validating Personal ID Documents

#### Example 1: Validate a Spanish NIF
```javascript
const { isValidIdDoc } = require('id-doc-validator');

// Validate a Spanish NIF (Número de Identificación Fiscal)
const isValid = isValidIdDoc('12345678Z', 'ES', 'nif');
console.log(isValid); // true or false
```

#### Example 2: Validate a German Passport
```javascript
const { isValidIdDoc } = require('id-doc-validator');

// Validate a German passport
const isValid = isValidIdDoc('C01X00T47', 'DE', 'passport');
console.log(isValid); // true or false
```

#### Example 3: Auto-detect document type
```javascript
const { isValidIdDoc } = require('id-doc-validator');

// Let the validator check all supported document types for Portugal
const isValid = isValidIdDoc('12345678', 'PT');
// Will try: CC, NIF, and Passport formats
console.log(isValid); // true if valid for any supported type
```

### Validating VAT Numbers

#### Example 4: Validate a VAT number (includes country code)
```javascript
const { isValidVat } = require('id-doc-validator');

// Validate a Spanish VAT number
const isValid = isValidVat('ESB12345678');
console.log(isValid); // true or false

// Validate a German VAT number
const isValidDE = isValidVat('DE123456789');
console.log(isValidDE); // true or false
```

#### Example 5: Validate VAT using VIES API (EU only)
```javascript
const { isValidViesVat } = require('id-doc-validator');

// Validate using the European Commission's VIES system
async function checkVat() {
  const result = await isValidViesVat('12345678', 'ES');
  console.log(result);
  // Returns:
  // {
  //   isValid: true/false,
  //   userError: 'VALID' | 'INVALID' | error code,
  //   vatNumber: '12345678'
  // }
}

checkVat();
```

### Getting Supported Countries and Document Types

#### Example 6: List all supported countries for ID documents
```javascript
const { supportedCountriesIdDoc } = require('id-doc-validator');

const countries = supportedCountriesIdDoc();
console.log(countries); // ['AT', 'BE', 'BG', 'BR', 'CA', 'CY', ...]
```

#### Example 7: List all supported document types for a specific country
```javascript
const { supportedIdDocsByCountry } = require('id-doc-validator');

const ptDocTypes = supportedIdDocsByCountry('PT');
console.log(ptDocTypes); // ['cc', 'nif', 'passport']

const esDocTypes = supportedIdDocsByCountry('ES');
console.log(esDocTypes); // ['nif', 'nie', 'passport']
```

#### Example 8: List all countries that support VAT validation
```javascript
const { supportedCountriesVat } = require('id-doc-validator');

const vatCountries = supportedCountriesVat();
console.log(vatCountries); // ['AT', 'BE', 'BG', 'BR', 'CO', 'CY', ...]
```

## 💡 Common Use Cases

### Form Validation
```javascript
const { isValidIdDoc } = require('id-doc-validator');

function validateUserIdDocument(idNumber, country, docType) {
  if (!idNumber || !country) {
    return { valid: false, message: 'ID number and country are required' };
  }

  const isValid = isValidIdDoc(idNumber, country, docType);
  
  return {
    valid: isValid,
    message: isValid ? 'Valid document' : 'Invalid document number'
  };
}

// Usage in a form
const result = validateUserIdDocument('12345678Z', 'ES', 'nif');
console.log(result); // { valid: true, message: 'Valid document' }
```

### Business Registration Validation
```javascript
const { isValidVat, isValidViesVat } = require('id-doc-validator');

async function validateBusinessVat(vatNumber, country) {
  // First, do a format validation (offline, fast)
  const formatValid = isValidVat(vatNumber);
  
  if (!formatValid) {
    return { valid: false, message: 'Invalid VAT format' };
  }

  // For EU countries, verify with VIES API (online, slower)
  if (['AT', 'BE', 'BG', 'CY', 'CZ', 'DE', 'DK', 'EE', 'ES', 'FI', 
       'FR', 'HR', 'HU', 'IE', 'IT', 'LT', 'LU', 'LV', 'MT', 'NL', 
       'PL', 'PT', 'RO', 'SE', 'SK', 'SL'].includes(country)) {
    try {
      const viesResult = await isValidViesVat(
        vatNumber.substring(2), // Remove country code
        country
      );
      
      return {
        valid: viesResult.isValid,
        message: viesResult.isValid ? 'VAT verified' : 'VAT not found in VIES',
        viesData: viesResult
      };
    } catch (error) {
      // Fallback to format validation if VIES is unavailable
      return { valid: formatValid, message: 'Format valid, VIES unavailable' };
    }
  }

  return { valid: true, message: 'Format valid' };
}
```

### Dynamic Country Selection
```javascript
const { supportedCountriesIdDoc, supportedIdDocsByCountry } = require('id-doc-validator');

// Get all countries for dropdown
const countries = supportedCountriesIdDoc();

// When user selects a country, show available document types
function getDocumentTypesForCountry(country) {
  const docTypes = supportedIdDocsByCountry(country);
  
  // Map to friendly names
  const friendlyNames = {
    'passport': 'Passport',
    'nif': 'NIF (Tax ID)',
    'nie': 'NIE (Foreigner ID)',
    'cc': 'CC (Citizen Card)',
    'gic': 'Identity Card',
    'cf': 'Codice Fiscale',
    'cni': 'Carte Nationale d\'Identité'
  };

  return docTypes.map(type => ({
    value: type,
    label: friendlyNames[type] || type.toUpperCase()
  }));
}

console.log(getDocumentTypesForCountry('ES'));
// [
//   { value: 'nif', label: 'NIF (Tax ID)' },
//   { value: 'nie', label: 'NIE (Foreigner ID)' },
//   { value: 'passport', label: 'Passport' }
// ]
```

## ⚠️ Important Notes

- **Country Codes**: Use ISO 3166-1 alpha-2 country codes (e.g., "ES", "FR", "DE")
- **Document Type**: Document type parameter is case-insensitive (e.g., "nif", "NIF", "Nif" all work)
- **VAT Country Codes**: Most VAT numbers use the same country code, except Greece (use "EL" instead of "GR") and Slovenia (use "SI" instead of "SL")
- **VIES API**: The European Commission's VIES API has rate limits and may be unavailable at times. Use it sparingly and implement fallback logic.
- **Validation Type**: This library validates the format and checksum of documents, not their actual existence or validity in official databases (except when using `isValidViesVat`)

## Supported Countries

<details>
<summary><strong>Austria (AT)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Belgium (BE)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Brazil (BR)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Bulgaria (BG)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Canada (CA)</strong></summary>

- Passport

</details>

<details>
<summary><strong>Colombia (CO)</strong></summary>

- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Czech Republic (CZ)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Croatia (HR)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Denmark (DK)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Estonia (EE)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Finland (FI)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>France (FR)</strong></summary>

- CNI (Carte Nationale d'Identité)
- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Germany (DE)</strong></summary>

- GIC (German Identity Card)
- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Greece (GR)</strong></summary>

- Passport
- VAT (Value Added Tax ID) (country code: EL)

</details>

<details>
<summary><strong>Hungary (HU)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Ireland (IE)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Italy (IT)</strong></summary>

- CF (Codice Fiscale)
- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Lithuania (LT)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Latvia (LV)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Luxembourg (LU)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Malta (MT)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Mexico (MX)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Netherlands (NL)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Poland (PL)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Portugal (PT)</strong></summary>

- CC (Cartão de Cidadão)
- NIF (Número de Identificação Fiscal)
- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Republic of Cyprus (CY)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Romania (RO)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Spain (ES)</strong></summary>

- DNI/NIF (Documento Nacional de Identidad / Número de Identificación Fiscal)
- NIE (Número de Identificación de Extranjero)
- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Slovakia (SK)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>Slovenia (SL)</strong></summary>

- Passport
- VAT (Value Added Tax ID) (country code: SI)

</details>

<details>
<summary><strong>Sweden (SE)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>United Kingdom (GB)</strong></summary>

- Passport
- VAT (Value Added Tax ID)

</details>

<details>
<summary><strong>United States (US)</strong></summary>

- Passport

</details>

## Installation

To use the `id-doc-validator` library in your project, you can install it via npm or yarn:

```bash
npm install id-doc-validator
# OR
yarn add id-doc-validator
```

## How to Use

### `isValidIdDoc`

To validate personal identification documents, use the `isValidIdDoc` function. It takes three parameters:

- `idDoc` (string): The identification document number to validate.
- `country` (string): The alpha-2 country code following ISO 3166-1 (e.g., "ES" for Spain, "FR" for France).
- `idDocType` (string, optional): The type of identification document to validate. For a list of supported identification document types, please refer to the expanded view of the [**Supported Countries**](#supported-countries) (to validate VAT, use [`isValidVat`](#isvalidvat)). If this parameter is not passed, the function will check if the passed id doc is valid for any of the supported id docs for the country.

### `isValidVat`

To validate any VAT number from the list of [**Supported Countries**](#supported-countries), use the `isValidVat` function. It takes one parameter:

- `vatNumber` (string): The VAT number to validate. Should include the VAT country code. In most cases it coincides with the alpha-2 country code, with some exceptions (e.g., "EL" for Greece instead of "GR").

### `isValidViesVat`

To validate a VAT number for an EU member state, use the `isValidViesVat` function. This function uses the API provided by the European Commission to validate the VAT number. It takes two parameters:

- `vatNumber` (string): The VAT number to validate. Should not include the country code.
- `countryCode` (string): The alpha-2 country code following ISO 3166-1 (e.g., "ES" for Spain, "FR" for France).

It returns an object with the following properties:

- `isValid` (boolean): Whether the VAT number is valid or not.
- `userError` (boolean): The error returned by the VIES API. If the request was successful, it will equal 'VALID' or 'INVALID'. If the request was not successful, it will return a string with the error code.
- `vatNumber` (string): The VAT number actually validated. For example, if the passed VAT number is "ES12345678", the returned VAT number will be "12345678", without the country code.

Please note that the VIES API is very limited in the number of requests it can handle. Please use moderately and expect the service to be unavailable at times.

### `supportedIdDocsByCountry`

To get a list of supported identification documents for a country, use the `supportedIdDocsByCountry` function. It takes one parameter:

- `country` (string): The alpha-2 country code following ISO 3166-1 (e.g., "ES" for Spain, "FR" for France).

It returns an array of strings with the supported identification documents for the country.

### `supportedCountriesIdDoc`

To get a list of supported countries for identification documents (not VAT), use the `supportedCountriesIdDoc` function. It takes no parameters.

It returns an array of strings with the supported countries.

### `supportedCountriesVat`

To get a list of supported countries for VAT validation, use the `supportedCountriesVat` function. It takes no parameters.

It returns an array of strings with the supported VAT country codes for VAT validation.

## Resources

- [ISO 3166-1](https://en.wikipedia.org/wiki/ISO_3166-1)
- [Consilium Europa - Check Document numbers](https://www.consilium.europa.eu/prado/en/check-document-numbers/check-document-numbers.pdf)
- [VIES VAT number validation](https://ec.europa.eu/taxation_customs/vies/#/vat-validation)
