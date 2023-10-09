---
title: "medrec: Regions form"
description: "Notes on approach to regions form"
date: "2023-10-09"
categories: ["Project"]
tags: ["medrec", "django", "htmx", "alpine.js", "daisyui"]
image: Chiyo_Mihama_With_Python_Homework.png
draft: true
---

## TLDR:

You can view the source code in [medrec](https://github.com/seyLu/medrec) repo.

## Django stuff

### API

For the api, I went with the structure:

```bash
<children>/?<parent>=<parent_code>
# eg. disctricts/?city=<city_code>
```

`urls.py`

```py
path("districts/", views.DistrictsQueryView.as_view(), name="districts-query"),
```

I used the built-in python library `urlib` to parse the url string, filter the data, then send back a json response.

`views.py`

```py
from urllib.parse import parse_qs, urlparse

class DistrictsQueryView(View):
    def post(self, request: HttpRequest) -> JsonResponse:
        parsed_url = urlparse(request.get_full_path())
        regions: dict[str, list[str]] = parse_qs(parsed_url.query)

        response: Any = []

        if code := regions.get("code"):
            code = code[0]  # type: ignore
            response = District.objects.get(code=code)

        elif city := regions.get("city"):
            city = city[0]  # type: ignore
            response = District.objects.filter(city=city)  # type: ignore[misc]

        if not response:
            return JsonResponse(list(response), safe=False)

        return JsonResponse(list(response.values()), safe=False)
```

### Response

Sample district json response:

```json
{
    "code": "082601001", // district code
    "name": "Aguinaldo",
    "region": "080000000", // this and below are foreign keys
    "province": "082600000",
    "city": " 082601000"
}
```

## Approach 1: HTMX & Datalist

Was not able to document the code. The changes happened on the same PR as approach 2, and I tend to sqaush my commits so yeah. The gist of it is that I used HTMX and datalist first. Then ran into some issues, one of it is saving state, and another one is how datalist works -- value is the text that is shown on the client, but that doesn't necessarily mean that that is what we want to send to the server.

## Approach 2: Alpine.js & Dropdown + filter

Then I tried alpine.js which was great at saving state, and instead of using datalist, used dropdown menu with a alpine template of unordered lists and a search filter.

I first worked on province form and handling which cities to query:

### Alpine.js saving state

```html
<div ...
     x-data="{
     provinces: [],

     provinceName: null,
     provinceCode: null,
     cityName: null,
     cityCode: null,
     ...
```

### Initializing provinces using Alpine.js

Using Alpine.js, fetch provinces from server:

```js
async getProvinces() {
    this.provinces = await (await fetch({% url 'provinces-query' %}, {
    method: 'POST',
    headers: {
        'Content-type': 'application/json; charset=UTF-8',
        'X-CSRFToken': $store.csrf_token,
    }, mode: 'same-origin'
    })).json()
},
```

We need to somehow call the getProvinces when we need the data:

```html
<div x-init="getProvinces" ...></div>
```

From the json response received, dynamically create html content, again, using Alpine.js:

```html
 <ul ...>
    <template x-for="item in provinces">
        <li @click="
            provinceName = item.name;
            provinceCode = item.code;
            @click.debounce="getCities">
            <a x-text="item.name"></a>
        </li>
    </template>
</ul>
```

### Adding search filter on user input

```html
 <div x-data="{
        search: '',

        get filteredProvinces() {
            return provinces.filter(
                province => province.name.toLowerCase().startsWith(this.search.toLowerCase())
            )
        },
     }"
     ...>
    <input x-model="search"
           id="province_name"
           name="province_name"
            ... />
    <ul ...>
        <template x-for="item in filteredProvinces" :key="item.name">
            <li @click="
                provinceName = item.name;
                provinceCode = item.code;
                @click.debounce="getCities">
            <a x-text="item.name"></a>
            </li>
        </template>
    </ul>
```

### Demo

And the output is something like this:

{{< video src="create_regions_form_using_alpinejs_and_dropdown_search.mp4" controls="yes" >}}
