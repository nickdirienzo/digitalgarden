---
{"dg-publish":true,"permalink":"/generating-example-numbers-with-faker-js-and-libphonenumber-js/"}
---

One of [[Mirage Security \| our interview questions]] is around formatting phone numbers. If you've worked around telephony at all, you know that this is not straightforward. 

For us, this problem is challenging because we sync enterprise directory data where formats are all over the place, country codes may or may not exist, and there might be nested numbers where one has preference.

So I built a dataset in support of this question to dive deep. How do you think about this problem? What options do we have when dealing with a gnarly dataset like this?

This isn't about how to answer those questions well, but I did find generating this dataset really interesting. And I learned that `libphonenumber-js` has a built-in function for generating example phone numbers: `getExampleNumber`.

We can mimic this messy data a couple of ways:
1. Using `faker.phone.number()`: by default, this uses the `"human"` format, which generates numbers like: "1-317-870-1507 x49566", "861.911.5285", and "(978) 485-5429", so we can provide cases around country codes and extensions being available or not.
2. Using `getExampleNumber` from `libphonenumber-js`: this lets us generate international phone numbers for arbitrary countries. For example:
```ts
import examples from 'libphonenumber-js/mobile/examples'
import { parsePhoneNumberFromString, getExampleNumber } from 'libphonenumber-js';

const countryCodes = ['US', 'GB', 'DE', 'IN', 'BR', 'JP', 'ZA', 'FR', 'AU', 'CA'];
const country = countryCodes[Math.floor(Math.random() * countryCodes.length)];
const newNumber = getExampleNumber(country, examples)?.formatInternational();
```

This is only a small part of the script that I built out for the interview problem, but I learned a thing or two so figured to write it down. I'm trying to get better at this learning in public thing.