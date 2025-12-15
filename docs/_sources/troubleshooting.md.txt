# Troubleshooting Guide

## Overview

This document provides a comprehensive guide to all error types that can be thrown by the Emissions API, along with detailed troubleshooting steps to resolve them.

---

## Table of Contents

1. [HTTP Status Codes Summary](#http-status-codes-summary)
2. [Error Response Format](#error-response-format)
3. [Bad Request Errors (400)](#bad-request-errors-400)
4. [Forbidden Errors (403)](#forbidden-errors-403)
5. [Not Found Errors (404)](#not-found-errors-404)
6. [Method Not Allowed Errors (405)](#method-not-allowed-errors-405)
7. [Internal Server Errors (500)](#internal-server-errors-500)
8. [Common Error Scenarios by API](#common-error-scenarios-by-api)
9. [Best Practices](#best-practices)

---

## HTTP Status Codes Summary

| Status Code | Error Type | Description |
|-------------|------------|-------------|
| 400 | Bad Request | Invalid input data, validation failures, malformed JSON |
| 403 | Forbidden | Access denied to specific resources |
| 404 | Not Found | Resource not found or user lacks permission |
| 405 | Method Not Allowed | HTTP method not supported for the endpoint |
| 500 | Internal Server Error | Server-side errors, contact administrator |

---

## Error Response Format

All errors follow a consistent JSON format:

```json
{
  "status": 400,
  "message": "Detailed error message explaining the issue"
}
```

---

## Bad Request Errors (400)

### 1. Invalid JSON Format

**Error Message:**

```
"Invalid type for field '<field_name>': expected a <type> but got <actual_type>"
```

**Causes:**

- Incorrect data type for a field (e.g., string instead of number)
- Malformed JSON structure
- Missing quotes around string values

**Troubleshooting Steps:**

1. Validate your JSON using a JSON validator
2. Check that all field types match the API specification:
   - `value`: Double or List<Double>
   - `date`: String in format "yyyy-MM-dd"
   - `country`: String (3-letter alpha code)
   - `includeDetails`: Boolean
3. Ensure proper JSON syntax (commas, brackets, quotes)

**Example Fix:**

```json
// ❌ Wrong
{
  "activity": {
    "value": "100",  // String instead of number
    "unit": kwh      // Missing quotes
  }
}

// ✅ Correct
{
  "activity": {
    "value": 100,
    "unit": "kwh"
  }
}
```

---

### 2. Unrecognized Field in Request

**Error Message:**

```
"Unrecognized field '<field_name>' found in request payload. Please verify your request."
```

**Causes:**

- Typo in field name
- Using a field not supported by the API
- Extra fields in the payload

**Troubleshooting Steps:**

1. Check the API documentation for correct field names
2. Remove any unsupported fields
3. Verify spelling and case sensitivity

**Example Fix:**

```json
// ❌ Wrong
{
  "activity": {
    "amount": 100,  // Should be "value"
    "units": "kwh"  // Should be "unit"
  }
}

// ✅ Correct
{
  "activity": {
    "value": 100,
    "unit": "kwh"
  }
}
```

---

### 3. Missing Required Parameters

**Error Message:**

```
"Missing required request parameter: <parameter_name>"
```

**Causes:**

- Required field not provided in the request
- Field is null or blank when it shouldn't be

**Troubleshooting Steps:**

1. Review the API documentation for required fields
2. Ensure all mandatory fields are present
3. Check that values are not null or empty strings

**Common Required Fields:**

- `activity.value`: Cannot be null
- `activity.unit`: Cannot be null or blank
- `activity.type` or `activity.factorId`: At least one must be provided
- `location.country`: Required if not using factorlink IDs
- `time.date`: Required if not using factorlink IDs

---

### 4. Invalid Date Format

**Error Message:**

```
"Invalid date format. Please provide a valid date in yyyy-MM-dd."
"Date must not be before 1900-01-01"
"Date <date> is not valid"
```

**Causes:**

- Incorrect date format
- Date before minimum allowed date (1900-01-01)
- Invalid date values (e.g., February 30)

**Troubleshooting Steps:**

1. Use the format: `yyyy-MM-dd` (e.g., "2022-01-15")
2. Ensure date is after 1900-01-01
3. Validate the date is a real calendar date

**Example Fix:**

```json
// ❌ Wrong
{
  "time": {
    "date": "01/15/2022"  // Wrong format
  }
}

// ✅ Correct
{
  "time": {
    "date": "2022-01-15"
  }
}
```

---

### 5. Invalid Location Data

**Error Messages:**

```
"<region> is not valid Country, please provide valid alpha 3 code."
"<region> is not valid StateProvince. Please provide valid state/province"
"<region> is not a region of given country"
"<powerGrid> is not a valid powerGrid"
"<powerGrid> powerGrid is duplicated"
"country can't be null or blank"
"stateProvince can't be null or blank"
```

**Causes:**

- Invalid country code (must be 3-letter ISO alpha-3 code)
- State/province doesn't exist or doesn't belong to the specified country
- Invalid power grid name
- Duplicate power grid entries in database

**Troubleshooting Steps:**

1. Use [ISO 3166-1 alpha-3 country codes](https://www.iso.org/iso-3166-country-codes.html) (e.g., "USA", "GBR", "JPN")
2. Verify state/province name matches the database
3. Check power grid name spelling and availability
4. Use either `stateProvince` OR `powerGrid`, not both
5. Query the `/carbon/<api>/area` endpoint to get valid locations

**Example Fix:**

```json
// ❌ Wrong
{
  "location": {
    "country": "US",  // Should be 3-letter code
    "stateProvince": "NY"  // Should be full name
  }
}

// ✅ Correct
{
  "location": {
    "country": "USA",
    "stateProvince": "New York"
  }
}

// ✅ Alternative with power grid
{
  "location": {
    "country": "USA",
    "powerGrid": "Zone ASCC Alaska Grid - AKGD"
  }
}
```

---

### 6. Invalid Unit of Measurement (UOM)

**Error Messages:**

```
"<uom> is not a valid unit. Please provide valid unit."
"<uom> unit is not supported for given type."
"input and output unit type does not match"
```

**Causes:**

- Unit not recognized by the system
- Unit incompatible with the data type
- Attempting to convert between incompatible unit types

**Troubleshooting Steps:**

1. Query the `/carbon/<api>/units` endpoint to get valid units
2. Ensure unit matches the activity type
3. Check for typos in unit names
4. Verify unit is supported for the specific API

**Example Fix:**

```json
// ❌ Wrong
{
  "activity": {
    "value": 100,
    "unit": "kilowatt-hour",  // Not recognized
    "type": "electricity"
  }
}

// ✅ Correct
{
  "activity": {
    "value": 100,
    "unit": "kwh",
    "type": "electricity"
  }
}
```

---

### 7. Invalid Data Type

**Error Messages:**

```
"Requested data type invalid/not supported for the API being called"
"Given dataType and subType is not valid. Please provide valid data"
```

**Causes:**

- Data type not supported by the specific API
- Typo in data type name
- Using a data type from a different API

**Troubleshooting Steps:**

1. Query the `/carbon/<api>/types` endpoint to get valid types
2. Verify the data type is supported by the API you're calling
3. Check spelling and case sensitivity

**Example Fix:**

```json
// ❌ Wrong - Using stationary type in location API
{
  "activity": {
    "value": 100,
    "unit": "mmbtu",
    "type": "Natural Gas"  // This is for stationary API
  }
}

// ✅ Correct - Using location type
{
  "activity": {
    "value": 100,
    "unit": "kwh",
    "type": "electricity"
  }
}
```

---

### 8. Invalid Factor ID

**Error Messages:**

```
"<factorId> is not a valid factorId"
"<factorId> factorId is unsupported for this API"
```

**Causes:**

- Factor ID doesn't exist in the database
- Factor ID not supported by the specific API

**Troubleshooting Steps:**

2. Use the `/carbon/factor/search` API to find valid factor IDs
3. Ensure the factor ID is supported by the API you're calling
4. Check that you have permission to access the factor

**Example:**

```json
// Valid factor ID format
{
  "activity": {
    "value": 100,
    "unit": "kwh",
    "factorId": "1234"
  }
}
```

---

### 9. Conflicting Input Parameters

**Error Messages:**

```
"Both factor ID and data type cannot be passed together"
"Either 'type' or 'factorId' must be provided in activity"
"Location not expected in request when factorId is present"
"Date not expected in request when factorId is present"
"FactorSet not expected in request when factorId is present"
"FactorVersion not expected in request when factorId is present"
"powerGrid is not supported for given API."
```

**Causes:**

- Providing both `type` and `factorId` in the same request
- Including location/date/factorSet when using factorId
- Using powerGrid with APIs that don't support it

**Troubleshooting Steps:**

1. Choose ONE approach:
   - **Option A**: Use `type` with location, date, factorSet (system finds the factor)
   - **Option B**: Use `factorId` only (factor already specifies everything)
2. Remove conflicting fields
3. Check API documentation for supported fields

**Example Fix:**

```json
// ❌ Wrong - Conflicting parameters
{
  "time": {"date": "2022-01-01"},
  "location": {"country": "USA"},
  "activity": {
    "value": 100,
    "unit": "kwh",
    "type": "electricity",
    "factorId": "123-456-789"  // Conflicts with type, location, date
  }
}

// ✅ Correct - Option A: Using type
{
  "time": {"date": "2022-01-01"},
  "location": {"country": "USA"},
  "activity": {
    "value": 100,
    "unit": "kwh",
    "type": "electricity"
  }
}

// ✅ Correct - Option B: Using factorId
{
  "activity": {
    "value": 100,
    "unit": "kwh",
    "factorId": "1234"
  }
}
```

---

### 10. Invalid Value/List

**Error Messages:**

```
"value List should not have more than 2 entries"
"Expected single value for given unit"
"should have atleast 1 and atmost 2 entries"
"must be greater than 0.0"
```

**Causes:**

- Providing more than 2 values in the value list
- Providing multiple values when unit expects single value
- Value is zero or negative

**Troubleshooting Steps:**

1. For simple units (kwh, kg, etc.): provide single value
2. For compound units (kwh/sqmt): provide 1-2 values
3. Ensure all values are positive numbers

**Example Fix:**
```json
// ❌ Wrong
{
  "activity": {
    "value": [100, 200, 300],
    "unit": "kwh"
  }
}

// ✅ Correct - Single value
{
  "activity": {
    "value": 100,
    "unit": "kwh"
  }
}

// ✅ Correct - Compound unit
{
  "activity": {
    "type": "Freight - Cargo Ship - Bulk Carrier - 0-9999 dwt",
	"unit": ["t","km"],
	"value": [1.0,1.0]
  }
}
```

---

### 11. Pagination Errors

**Error Messages:**

```
"Page size <size> is exceeding the maximum allowed size of 30"
"Either Page Number or Size is below 0"
```

**Causes:**

- Page size exceeds maximum limit (30)
- Negative page number or size

**Troubleshooting Steps:**

1. Keep page size ≤ 30
2. Use page numbers ≥ 0

**Example Fix:**

```json
// ❌ Wrong
{
  "pagination": {
    "page": -1,
    "size": 50
  }
}

// ✅ Correct
{
  "pagination": {
    "page": 0,
    "size": 20
  }
}
```

---

### 12. Validation Constraint Violations

**Error Message:**

```
"<field_path>: <validation_message>"
```

**Causes:**

- Field fails validation constraints (e.g., @NotNull, @NotBlank, @Min, @Max)
- Multiple validation failures shown as comma-separated list

**Troubleshooting Steps:**

1. Check each field mentioned in the error
2. Ensure required fields are not null or blank
3. Verify numeric values are within allowed ranges
4. Review API documentation for field constraints

---

## Forbidden Errors (403)

### Access Denied to Factor

**Error Message:**

```
"Access denied, User does not have permission to access factorId: <factorId>"
```

**Causes:**

- User doesn't have permission to access the specified factor

**Troubleshooting Steps:**

1. Verify you have the correct permissions
3. Contact your administrator to request access
4. Use the `/carbon/factor/search` API to find factors you can access

---

## Not Found Errors (404)

### 1. Resource Not Found

**Error Messages:**

```
"Emission Record not found"
"Factor Records not found"
"Resource not found at /<path>"
"<resource> either does not exist, or user does not have permission to access"
```

**Causes:**

- No matching emission factors found for the given criteria
- Invalid API endpoint
- User lacks permission to access the resource
- Data doesn't exist in the database for the specified parameters

**Troubleshooting Steps:**

1. Verify the API endpoint URL is correct
2. Check that data exists for your location/date/type combination
3. Try broadening your search criteria
4. Use the search API to find available factors
5. Verify you have permission to access the resource
6. Check if the factor set has data for the specified time period

**Example:**

```json
// If you get "Emission Record not found" for:
{
  "time": {"date": "2025-01-01"},  // Future date
  "location": {"country": "XXX"},   // Invalid country
  "activity": {
    "value": 100,
    "unit": "kwh",
    "type": "electricity"
  }
}
```

---

## Method Not Allowed Errors (405)

**Error Message:**

```
"Method <HTTP_METHOD> is not supported for this request. Supported methods are: <methods>"
```

**Causes:**

- Using wrong HTTP method (e.g., GET instead of POST)
- Endpoint doesn't support the HTTP method used

**Troubleshooting Steps:**

1. Check API documentation for correct HTTP method
2. Most calculation endpoints use POST
3. Metadata endpoints (types, units, area) typically use GET

**Supported Methods by Endpoint:**

- `/carbon/factor/search`: **POST**
- `/carbon/<api>/types`: **GET**
- `/carbon/<api>/units`: **GET**
- `/carbon/<api>/area`: **GET**
- `/carbon/factorset`: **GET**

---

## Internal Server Errors (500)

### General Internal Error

**Error Message:**

```
"Internal Server Error: please Contact Administrator"
```

**Causes:**

- Unexpected server-side error
- Database connection issues
- Data integrity problems
- Unit conversion failures
- Null pointer exceptions

**Troubleshooting Steps:**

1. Check server logs for detailed error information
2. Verify database connectivity
3. Ensure all required data is properly loaded
4. Contact system administrator with:
   - Timestamp of the error
   - Request payload
   - API endpoint called
   - Transaction ID (if available)

**Common Internal Error Scenarios:**

- **Unit Conversion Errors**: Incompatible units, unsupported metric prefix
- **Data Hierarchy Errors**: "Unable to calculate the Emission hierarchy"
- **Database Errors**: Connection timeout, query failures
- **Null Values**: Missing required data in database

---

## Common Error Scenarios by API

### Location API (`/carbon/location`)

**Common Errors:**

1. Invalid country code (must be 3-letter alpha-3)
2. State/province not found or doesn't match country
3. Power grid not valid or duplicated
4. Invalid type
5. Date outside available data range

**Supported Types:**

- electricity
- steam
- chilled water
- hot water

---

### Stationary API (`/carbon/stationary`)

**Common Errors:**

1. Invalid fuel type
2. Unit not in allowed list
3. Missing location data

---

### Mobile API (`/carbon/mobile`)

**Common Errors:**

1. Invalid type
2. Incompatible unit for vehicle type
3. Missing location data

---

### Fugitive API (`/carbon/fugitive`)

**Common Errors:**

1. Invalid refrigerant type
2. Unsupported unit for fugitive emissions
3. Missing location data

---

### Factor API (`/carbon/factor`)

**Common Errors:**

1. User doesn't have permission to access factor
2. Conflicting parameters (factorId with location/date)

---

### Search API (`/carbon/factor/search`)

**Common Errors:**

1. Invalid search criteria
2. No results found for search parameters
3. Pagination errors

---

### Calculation API (`/carbon/calculation`)

**Common Errors:**

1. Invalid data type for generic calculation
2. Missing required activity data
3. Unsupported unit for calculation type

---

## Best Practices

### 1. Input Validation
- Always validate your JSON before sending requests
- Use the correct data types for each field
- Ensure dates are in yyyy-MM-dd format
- Use 3-letter ISO alpha-3 country codes

### 2. Error Handling
- Implement proper error handling in your client code
- Log error responses for debugging
- Parse the error message for specific guidance
- Don't retry on 400-level errors without fixing the input

### 3. API Discovery
- Use metadata endpoints to discover valid values:
  - `/carbon/<api>/types` - Get valid data types
  - `/carbon/<api>/units` - Get valid units
  - `/carbon/<api>/area` - Get valid locations
  - `/carbon/factorset` - Get available factor sets

### 4. Testing
- Test with known good data first
- Validate one field at a time when debugging
- Use the examples in API documentation
- Check response status codes and messages

### 5. Performance
- Use pagination for large result sets
- Cache metadata (types, units, locations) in your application
- Batch requests when possible
- Set appropriate page sizes (≤30)

### 6. Security
- Ensure you have proper permissions for factors
- Validate user access before making requests