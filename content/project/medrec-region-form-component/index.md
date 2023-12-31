---
title: "Medrec Region Form Component"
description: "Trying out HTMX/Alpine.js on a form component"
date: "2023-10-17"
categories: ["Project"]
tags: ["medrec", "django", "htmx", "alpine.js", "daisyui", "tailwindcss"]
image: Chiyo_Mihama_With_Python_Homework.png
---

You can view the source code in [medrec](https://github.com/seyLu/medrec) repo. Final changes were made using [Approach 2](#approach-2) and the code changes are on [this PR](https://github.com/seyLu/medrec/pull/17/files).

## Django

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

<!-- prettier-ignore-start -->
```json
{
  "code": "082601001", // district code
  "name": "Aguinaldo",
  "region": "080000000", // this and below are foreign keys
  "province": "082600000",
  "city": " 082601000"
}
```
<!-- prettier-ignore-end -->

## Approach 1: Alpine.js & Dropdown + Filter

Then I tried alpine.js which was great at saving state, and instead of using datalist, used dropdown menu with a alpine template of unordered lists and a search filter.

I first worked on province form and handling which cities to query:

### Alpine.js Saving State

<!-- prettier-ignore-start -->
```html
<div
  x-data="{
    provinces: [],

    provinceName: null,
    provinceCode: null,
    cityName: null,
    cityCode: null,
```
<!-- prettier-ignore-end -->

### Initializing Provinces

Using Alpine.js, fetch provinces from server:

<!-- prettier-ignore-start -->
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
<!-- prettier-ignore-end -->

We need to somehow call the getProvinces when we need the data:

```html
<div x-init="getProvinces"></div>
```

From the json response received, dynamically create html content, again, using Alpine.js:

<!-- prettier-ignore-start -->
```html
<ul>
  <template x-for="item in provinces">
    <li>
        <a x-text="item.name"></a>
    </li>
  </template>
</ul>
```
<!-- prettier-ignore-end -->

### Search Filter On User Input

<!-- prettier-ignore-start -->
```html
<div
  x-data="{
    search: '',

    get filteredProvinces() {
      return provinces.filter((province) => {
          province.name
            .toLowerCase()
            .startsWith(this.search.toLowerCase())
      })
    },
  }"
>
  <input x-model="search" id="province_name" name="province_name" />
  <ul>
    <template x-for="item in filteredProvinces" :key="item.name">
      <li>
        <a x-text="item.name"></a>
      </li>
    </template>
  </ul>
</div>
```
<!-- prettier-ignore-end -->

### Fetching Cities

To fetch cities, added onclick handler on list item and added debounce to minimize calls to server:

<!-- prettier-ignore-start -->
```html
<template x-for="item in filteredProvinces" :key="item.name">
  <li
    @click.debounce="getCities"
    @click="
      provinceName = item.name;
      provinceCode = item.code;
    "
  >
    <a x-text="item.name"></a>
  </li>
</template>
```
<!-- prettier-ignore-end -->

Then added `getCities` logic:

<!-- prettier-ignore-start -->
```js
async getCities() {
    this.cities = await (await fetch(`{% url 'cities-query' %}?province=${this.provinceCode}`, {
    method: 'POST',
    headers: {
        'Content-type': 'application/json; charset=UTF-8',
        'X-CSRFToken': $store.csrf_token,
    }, mode: 'same-origin'
    })).json()
},
```
<!-- prettier-ignore-end -->

### Demo

And the output is something like this:

{{< video src="/project/medrec-region-form-component/create_regions_form_using_alpinejs_and_dropdown_search.mp4" controls="yes" >}}

If you're curious about all the changes made using this approach, you can [view this PR](https://github.com/seyLu/medrec/pull/11/files).

<div id="approach-2"></div>

## Approach 2: HTMX & Datalist

### Handling HTMX On The Backend

Check if request comes from htmx or not and respond to client accordingly (htmx expects html):

```py
class ProvincesQueryView(View):
    def post(self, request: HttpRequest) -> JsonResponse | HttpResponse:
        parsed_url = urlparse(request.get_full_path())
        regions: dict[str, list[str]] = parse_qs(parsed_url.query)

        response: Any = []

        if code := regions.get("code"):
            code = code[0]  # type: ignore
            response = Province.objects.get(code=code)

        else:
            response = Province.objects.all()

        if not request.htmx:
            if not response:
                return JsonResponse(list(response), safe=False)

            return JsonResponse(list(response.values()), safe=False)

        if not response:
            return HttpResponse()

        return render(
            request,
            "medrec/partials/regions-form/province-datalist.html",
            {"provinces": list(response)},
        )
```

And added templates to send to client:

<!-- prettier-ignore-start -->
```html
{% for province in provinces %}
  <option value="{{ province.name }}" data-code="{{ province.code }}">
    {{ province.code }}
  </option>
{% endfor %}
```
<!-- prettier-ignore-end -->

### HTMX Swap

On datalist element load, request provinces from server and swap the template inside the datalist element:

<!-- prettier-ignore-start -->
```html
<input
  type="text"
  id="province_name"
  name="province_name"
  list="province_datalist"
  required
  placeholder="Leyte"
  class="input input-bordered w-full"
/>
<datalist
  hx-post="{% url 'provinces-query' %}"
  hx-trigger="load"
  hx-target="this"
  hx-swap="innerHTML"
  id="province_datalist"
></datalist>
```
<!-- prettier-ignore-end -->

### Handling Datalist Option On Click

The idea was, once a province option was clicked, it would trigger a request to the server for cities under it. The problem was, datalists don't really support most javascript events that normal html elements do, including onclick.

One way to circumvent this is to add an oninput change handler, then detect if it was a mouse click.

<!-- prettier-ignore-start -->
```js
const provinceName = document.getElementById('province_name');
const cityName = document.getElementById('city_name');
const districtName = document.getElementById('district_name');

const province = document.getElementById('province');
const provinceDatalist = document.getElementById('province_datalist');
let provinceNameEsrc = null;

provinceName.addEventListener('keydown', (e) => {
    provinceNameEsrc = e.key ? 'input' : 'list';
});

provinceName.addEventListener('input', (e) => {
    if (provinceNameEsrc === 'list') {
        const val = e.target.value;
        const province_code = provinceDatalist.querySelector(
            `option[value="${val}"]`
        ).dataset.code;

        province.value = province_code;
        province.setAttribute(
            'hx-post',
            `{% url 'cities-query' %}?province=${province_code}`
        );
        htmx.process(province);

        province.dispatchEvent(new Event('change'));
        cityName.disabled = false;
        cityName.value = '';
    } else {
        cityName.disabled = true;
        cityName.value = '';
        districtName.disabled = true;
        districtName.value = '';
    }
});
```
<!-- prettier-ignore-end -->

I've used `htmx.process(<elem>)` after adding htmx attributes, so that once this elem is swapped, htmx would work as usual.

As for the custom `change` event, I used a hidden input elem to catch that event. This input elem stores `province_code`, and if the `province_code` changes, will trigger a request to the server to give back the cities under this province.

<!-- prettier-ignore-start -->
```html
<input
  type="hidden"
  id="province"
  name="province"
  hx-trigger="change"
  hx-target="#city_datalist"
  hx-swap="innerHTML"
/>
```
<!-- prettier-ignore-end -->

### Demo

{{< video src="/project/medrec-region-form-component/create_regions_form_using_htmx_and_datalist.mp4" controls="yes" >}}

You can [view this PR](https://github.com/seyLu/medrec/pull/17/files) to see the full changes made using this approach.
