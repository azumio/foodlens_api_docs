
# Azumio Food Recognition API V1

Last updated: 06/23/2021

The Food API takes a food photo and returns a list of most likely foods together with it's nutritional information.

## Photo requirements

- JPEG 544x544px, approximate size 50-70kb (set the jpeg encoder quality setting to this size on an average photo)
- Food in center and filling the frame.
- The photo should not be upscaled by the client to match the 544x544px requirenment

## API Calls

Make a POST to `https://api2.azumio.com/v1/foodrecognition/full` with the photo as `Content-Type` `multipart/form-data` or `image/jpeg`
together with your user_key. All calls MUST be made over https.

The **top** parameter disables grouping and limits the number of returned results.

Example API call:

    curl --request POST  -H "Content-Type: image/jpeg" --data-binary @apple.jpg "https://api2.azumio.com/v1/foodrecognition/full?top=5&user_key=abcd"
    
Alternative authentication: 

To keep the user_key our of the url and out of logs, use http basic authentication with username user_key and your user key as the password. (this methods is only available for customers with custom contract and is not available for keys from https://dev.caloriemama.ai/ )

    curl --request POST --user user_key:abdc -H "Content-Type: image/jpeg" --data-binary @apple.jpg "https://api2.azumio.com/v1/foodrecognition/full?top=5"

Response: `Content-Type: application/json`

    {
      "is_food": true,
      "lang": "en_US",
      "top_results": [
        {
          "food_id": "d0ea47a9fda5c37e",
          "group": "Pastry",
          "name": "Pretzel",
          "name[ko_KR]": "\ud504\ub808\uccbc",
          "nutrition": {
            "calcium": 0.00022999999999999998,
            "calories": 3380,
            "dietaryFiber": 0.017,
            "iron": 3.9200000004000005e-05,
            "monounsaturatedFat": 0.010709999999999999,
            "polyunsaturatedFat": 0.00948,
            "potassium": 0.00088,
            "protein": 0.082,
            "saturatedFat": 0.00695,
            "sodium": 0.00545,
            "sugars": 0.0025,
            "totalCarbs": 0.6939,
            "totalFat": 0.031
          },
          "score": 98,
          "servingSizes": [
            {
              "servingWeight": 0.143,
              "unit": "1 large"
            },
            {
              "servingWeight": 0.115,
              "unit": "1 medium"
            },
            {
              "servingWeight": 0.062,
              "unit": "1 small"
            },
            {
              "servingWeight": 0.1,
              "unit": "100 g"
            },
            {
              "servingWeight": 0.001,
              "unit": "1 g"
            },
            {
              "servingWeight": 0.0283495,
              "unit": "1 oz"
            }
          ]
        },
        ...
      ]
    }






### Result fields

**is_food** true if the photo is likely a photo of food
**lang** language of the response (might differ from the requested language if the language was not available)

#### top_results[]

**food_id** Unique ID of the food item
**score** The confidence score of the results. Where >= 100 is a VERY GOOD match and 50 is a MAYBE
**group** High level group of the food item (localized)
**name** Name of the food item (localized)
**name[..]** Additional localized names requested by name_lang
**nutrition** See Nutrition
**servingSizes** See Serving Sizes
**thumbnail** url to the thumbnail (only included if the **thumbnail** parameter is specified)

#### Nutrition
Nutritional information normalized to one unit of the food item. Usually this is 1kg however food items without servingWeight are not normalized.
To get the absolute nutritional values multiply it by the **servingWeight** from the **servingSizes** field

All values (including vitamins) are in kg.

### Serving Sizes
**servingWeight** Total weight for this serving size in kg
**unit** Serving size unit for display (localized)

## Localization

Response language or the user facing names can be requested by setting the Accept-Language HTTP header or passing in the **lang** query parameter.
The value to the header or lang parameter MUST be according to https://tools.ietf.org/html/rfc7231#section-5.3.5
The API will fallback to the language closest to the requested language or en_US, if no match is found.

Example:

    curl --request POST  -H "Content-Type: image/jpeg" --data-binary @apple.jpg "https://api2.azumio.com/v1/foodrecognition/full?top=5&lang=ko_KR&user_key=abcd"


The **lang** parameter takes precedence over the Accept-Language header.

### Requesting food names in additional languages

If required,  additional food name translations can we requested by specifying a **name_lang** parameter.
Multiple translations can be requested by passing in a list of languages.

Example CALL

    curl --request POST  -H "Content-Type: image/jpeg" --data-binary @apple.jpg "https://api2.azumio.com/v1/foodrecognition/full?top=5&name_lang=ko_KR,en_US&user_key=abcd"

Response:
    "name": "Pretzel",
    "name[en_US]": "Pretzel",
    "name[ko_KR]": "\ud504\ub808\uccbc",

## Requesting thumbnails

To request thumbnails include a **thumbnail** parameter with the requested thumbnail size.

Example:

    curl --request POST  -H "Content-Type: image/jpeg" --data-binary @apple.jpg "https://api2.azumio.com/v1/foodrecognition/full?top=5&thumbnail=128&user_key=abcd"


## API Errors

The API may return a HTTP status code other that 200 in case of an error.
All errors SHOULD have a detailed explanation encoded as application/json

Example:

    {
      "error": {
        "code": 400,
        "errorDetail": "lang=es is not supported"
      }
    }
