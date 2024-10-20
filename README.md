# Shopify-swatches
>Shopify version Dawn Theme 15
We're back with a new version of our collection swatches! We've updated the collection swatches.

Compatible Themes: This code should work on all free Shopify themes (Dawn, Refresh, Craft, Studio, Publisher, Crave, Origin, Taste, Colorblock, Sense, Ride, Spotlight).

>1. Prerequisite: Product Swatch v4 (Create Metaobjects and Metafields)
Create Metaobject “Variant Swatch Map”
Make sure handles of the metaobject and fields match those shown in the video since the code references those exact handles.

- Field 1: Title
Regular Expression: ```^[a-zA-Z0-9_-]+$``` (pattern to match - alphanumeric with - and _)

- Field 2: Variant Images JSON
The JSON will have the following schema defined:
```JSON
{
  "$id": "variants_images.schema.json",
  "$schema": "<http://json-schema.org/draft-07/schema#>",
  "title": "Variants List",
  "description": "A list of variant swatches",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "variant_name": {
        "type": "string",
        "description": "The variant option name."
      },
      "variant_value": {
        "type": "string",
        "description": "The variant option value."
      },
      "variant_swatch": {
        "type": "string",
        "description": "The filename or URL of the image representing the color."
      },
      "variant_hex": {
        "type": "string",
        "description": "The HEX value or description of the color."
      }
    },
    "required": [
      "variant_name",
      "variant_value"
    ]
  }
}

```
- Create Metaobject entry called “variant-swatch-mapping”
Add JSON entries that reference your uploaded images files. For example:
```JSON
[
  {
    "variant_name": "Material",
    "variant_value": "Cotton",
    "variant_swatch": "cotton.jpg"
  },
  {
    "variant_name": "Material",
    "variant_value": "Polyester",
    "variant_swatch": "polyester.jpg"
  },
  {
    "variant_name": "Color",
    "variant_value": "Blue",
    "variant_swatch": "bluerose.jpg"
  },
  {
    "variant_name": "Color",
    "variant_value": "Green",
    "variant_swatch": "greenimage.jpg"
  },
  {
    "variant_name": "Color",
    "variant_value": "Black",
    "variant_swatch": "",
    "variant_hex": "#000000"
  },
  {
    "variant_name": "Color",
    "variant_value": "Red",
    "variant_swatch": "",
    "variant_hex": "#FF0000"
  }
]

```

**Create metafield Variant Swatch Map Override**
Use type Variant Swatch Map.

This metafield is for any products you wish to have product-specific swatch mapping. It will override the default metaobject entry variant-swatch-mapping

### 2. Admin Preparation
>Turn off “Show second image on hover” in theme editor for all sections where card swatches are being used
>Collection page product grid
>Featured Collections
>Search
>Related Products
>Set the featured product image to the first variant
>Assign the featured product image to be the same as the first variant image to prevent any product card image mismatches with the variant image.

# Here the actual implementation start here for collection page swatches

## 3. Add new theme settings
- Edit  **settings_schema.json**
  - Inside these file you need to find out with ctrl+f press then search for cards.
  - After finding these code at the end of these json code you paste below code.
```JSON
  {
    "name": "Product Card Swatches",
    "settings": [
      {
        "type": "checkbox",
        "id": "card_swatches",
        "default": false,
        "label": "Enable Product Card Swatches"
      },
      {
        "type": "checkbox",
        "id": "card_swatch_lazy_load",
        "default": false,
        "label": "Lazy Load Variant Images"
      },
      {
        "type": "text",
        "id": "card_variant_option_name",
        "label": "Variant Option Name",
        "default": "Color"
      },
      {
        "type": "text",
        "id": "card_variant_swatch_metaobject",
        "label": "Default Variant Swatch Map Metaobject",
        "default": "variant-swatch-mapping",
        "info": "Can be overridden with product metafield"
      },
      {
        "id": "card_swatch_shape_custom",
        "label": "Swatch Shape",
        "type": "select",
        "options": [
          {
            "value": "circle",
            "label": "t:sections.main-product.blocks.variant_picker.settings.swatch_shape.options__1.label"
          },
          {
            "value": "square",
            "label": "t:sections.main-product.blocks.variant_picker.settings.swatch_shape.options__2.label"
          }
        ],
        "default": "circle"
      },
      {
        "type": "range",
        "id": "card_swatch_size_custom",
        "min": 10,
        "max": 40,
        "step": 1,
        "unit": "px",
        "label": "Swatch Size",
        "default": 25
      }
    ]
  },

```
4. Create New Snippet And Asset Files
Create liquid snippet card-variant-swatch-custom.liquid
```liquid
{% comment %}
  Description:
  Renders product variant swatch options for collection page based on image URLs. 
  Defaults to buttons if no image URL is present.

  Accepts:
  - product: {Object} product object.
  - option: {Object} current product_option object.
  - variant_images_data: list of variant images
  - lazy_load_all_variants: lazy load all variant images for immediate availability when clicking swatches

  Usage:
  {% render 'variant-swatch-collection',
    product: product,
    option: option,
    variant_images_data: variant_images_data,
    lazy_load_all_variants: lazy_load_all_variants
  %}
{% endcomment %}

{% assign base_store_files_url = '//' | append: shop.permanent_domain | append: '/cdn/shop/files/' %}
{% assign product_form_id = 'product-form-' | append: product.id %}

<div class="collection-product-card__swatch-variants">
  {% for value in option.values %}
    {% assign variant_image_url = nil %}
    {% assign variant_id = nil %}
    {% assign option_disabled = true %}
    {% assign first_match_found = false %}

    {% for variant in product.variants %}
      {% if first_match_found == false %}
        {% case option.position %}
          {% when 1 %}
            {% if variant.option1 == value %}
              {% assign variant_image_url = variant.featured_media | img_url: '360x360' %}
              {% assign variant_id = variant.id %}
              {% if lazy_load_all_variants %}
                <img 
                  src="{{ variant_image_url }}" 
                  alt="{{ variant.title }}" 
                  style="display: none;" 
                  loading="lazy"
                  data-variant-id="{{ variant.id }}"
                  width="{{ variant.featured_media.width }}"
                  height="{{ variant.featured_media.height }}">
              {% endif %}
              {% if variant.available %}
                {% assign option_disabled = false %}
              {% endif %}
              {% assign first_match_found = true %}
            {% endif %}
          {% when 2 %}
            {% if variant.option2 == value and variant.option1 == product.selected_or_first_available_variant.option1 %}
              {% assign variant_image_url = variant.featured_media | img_url: '360x360' %}
              {% assign variant_id = variant.id %}
              {% if lazy_load_all_variants %}
                <img 
                  src="{{ variant_image_url }}" 
                  alt="{{ variant.title }}" 
                  style="display: none;" 
                  loading="lazy"
                  data-variant-id="{{ variant.id }}"
                  width="{{ variant.featured_media.width }}"
                  height="{{ variant.featured_media.height }}">
              {% endif %}
              {% if variant.available %}
                {% assign option_disabled = false %}
              {% endif %}
              {% assign first_match_found = true %}
            {% endif %}
          {% when 3 %}
            {% if variant.option3 == value and variant.option1 == product.selected_or_first_available_variant.option1 and variant.option2 == product.selected_or_first_available_variant.option2 %}
              {% assign variant_image_url = variant.featured_media | img_url: '360x360' %}
              {% assign variant_id = variant.id %}
              {% if lazy_load_all_variants %}
                <img 
                  src="{{ variant_image_url }}" 
                  alt="{{ variant.title }}" 
                  style="display: none;" 
                  loading="lazy"
                  data-variant-id="{{ variant.id }}"
                  width="{{ variant.featured_media.width }}"
                  height="{{ variant.featured_media.height }}">
              {% endif %}           
              {% if variant.available %}
                {% assign option_disabled = false %}
              {% endif %}
              {% assign first_match_found = true %}
            {% endif %}
        {% endcase %}
      {% endif %}

      {% if variant_image_url %}
        {% assign swatch_found = false %}
        {% for item in variant_images_data %}
          {% if item.variant_value == value %}
            {% if item.variant_swatch != blank %}
              {% assign variant_image_url = base_store_files_url | append: item.variant_swatch %}
              {% assign swatch_found = true %}
            {% elsif item.variant_hex %}
              {% assign hex_color = item.variant_hex | replace: '#', '%23' %}
              {% assign svg = '<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40"><rect width="100%" height="100%" fill="' | append: hex_color | append: '" /></svg>' %}
              {% assign encoded_svg = svg | replace: '"', '%22' | replace: '<', '%3C' | replace: '>', '%3E' %}
              {% assign variant_image_url = 'data:image/svg+xml;charset=utf-8,' | append: encoded_svg %}
              {% assign swatch_found = true %}
            {% endif %}
            {% if swatch_found %}
              {% break %}
            {% endif %}
          {% endif %}
        {% endfor %}
      {% endif %}
    {% endfor %}

    {%- capture input_id -%}
      collection-{{ section.id }}-{{ product.id }}-{{ option.position }}-{{ forloop.index0 }}
    {%- endcapture -%}

    {%- capture input_name -%}
      collection-{{ section.id }}-{{ product.id }}-{{ option.name }}-{{ option.position }}
    {%- endcapture -%}

    <div class="collection-product-card__swatch">
      <input 
        type="radio"
        id="{{ input_id }}"
        name="{{ input_name }}"
        value="{{ value | escape }}"
        form="{{ product_form_id }}"
        data-section-id="{{ section.id }}"
        data-product-id="{{ product.id }}"
        data-variant-id="{{ variant_id }}"
        data-product-url="{{ product.url }}"
        data-option-value-id="{{ value.id }}"
        data-image-url="{{ variant_image_url }}"
        {% if option.selected_value == value %}
          checked
        {% endif %}
        {% if option_disabled %}
          class="disabled"
        {% endif %}
      >
      <label for="{{ input_id }}" style="background-image: url('{{ variant_image_url }}');">
        <span class="visually-hidden">{{ value | escape }}</span>
        <span class="visually-hidden">{{ 'products.product.variant_sold_out_or_unavailable' | t }}</span>
      </label>
    </div>
  {% endfor %}
</div>
```
- Create asset ```card-product-variant-selection-custom.js``` And Paste below code
```js
  document.addEventListener('DOMContentLoaded', function() {
    function updateProductImage(card, variantData, variantId) {
        var productImageElement = card.querySelector('.card__media img');
        if (productImageElement) {
            var lazyLoadAllVariants = card.getAttribute('data-lazy-load-all-variants') === 'true';
            if (lazyLoadAllVariants) {
                var preloadedImage = card.querySelector(`img[data-variant-id="${variantId}"]`);
                if (preloadedImage) {
                    productImageElement.src = preloadedImage.src;
                    productImageElement.srcset = preloadedImage.srcset;
                }
            } else if (variantData && variantData.imageUrl) {
                var dynamicSrcset = [
                    `${variantData.imageUrl}?width=165 165w`,
                    `${variantData.imageUrl}?width=360 360w`,
                    `${variantData.imageUrl}?width=533 533w`,
                    `${variantData.imageUrl}?width=720 720w`,
                    `${variantData.imageUrl}?width=940 940w`,
                    `${variantData.imageUrl}?width=1066 1066w`
                ].join(', ');
                productImageElement.srcset = dynamicSrcset;
                productImageElement.src = variantData.imageUrl;
            }
        }
    }

    function updateProductLinks(card, variantData) {
        var productLinks = card.querySelectorAll('a[id^="CardLink-"], a[id^="StandardCardNoMediaLink-"], a.card__content');
        productLinks.forEach(function(link) {
            link.href = variantData.productUrl;
        });

        if (variantData.productUrl) {
            var quickAddButton = card.querySelector('.quick-add__submit');
            if (quickAddButton) {
                quickAddButton.setAttribute('data-product-url', variantData.productUrl);
            }
        }
    }

    function storeSelectedVariant(sectionId, productId, variantId) {
        sessionStorage.setItem('selectedVariant-' + sectionId + '-' + productId, variantId);
    }

    function storeSelectedSwatch(sectionId, productId, variantId) {
        sessionStorage.setItem('selectedSwatch-' + sectionId + '-' + productId, variantId);
    }

    function restoreSelectedVariant(productGrid, sectionId, productId, variantDataMap) {
        var selectedVariantId = sessionStorage.getItem('selectedVariant-' + sectionId + '-' + productId);
        if (selectedVariantId && variantDataMap[selectedVariantId]) {
            var card = productGrid.querySelector(`.card-product-custom-div[data-section-id="${sectionId}"][data-product-id="${productId}"]`);
            var variantData = variantDataMap[selectedVariantId];
            updateProductImage(card, variantData, selectedVariantId);
            updateProductLinks(card, variantData);
        }
    }

    function restoreSelectedSwatch(productGrid, sectionId, productId) {
        var selectedVariantId = sessionStorage.getItem('selectedSwatch-' + sectionId + '-' + productId);
        if (selectedVariantId) {
            var swatchInput = productGrid.querySelector(`input[type="radio"][data-variant-id="${selectedVariantId}"][data-section-id="${sectionId}"][data-product-id="${productId}"]`);
            if (swatchInput) {
                swatchInput.checked = true;
            }
        }
    }

    function initializeProductGrid(productGrid) {
        var sectionId = productGrid.getAttribute('data-id');
        productGrid.querySelectorAll('.card-product-custom-div').forEach(function(card) {
            var productId = card.getAttribute('data-product-id');
            var variantDataMap = window['variantDataMap' + sectionId.replace(/-/g, '_') + '_' + productId];
            restoreSelectedVariant(productGrid, sectionId, productId, variantDataMap);
            restoreSelectedSwatch(productGrid, sectionId, productId);
            card.addEventListener('change', function(e) {
                if (e.target.matches('input[type="radio"][data-section-id="' + sectionId + '"][data-product-id="' + productId + '"]')) {
                    var variantId = e.target.getAttribute('data-variant-id');
                    var variantData = variantDataMap[variantId];
                    if (!variantData) {
                        return;
                    }
                    storeSelectedVariant(sectionId, productId, variantId);
                    storeSelectedSwatch(sectionId, productId, variantId);
                    updateProductImage(card, variantData, variantId);
                    updateProductLinks(card, variantData);
                }
            });
            card.classList.add('loaded');
        });
    }

    function initializeAllProductGrids() {
        var productGrids = document.querySelectorAll('.grid.product-grid');
        productGrids.forEach(initializeProductGrid);
    }

    initializeAllProductGrids();

    var observer = new MutationObserver(function(mutations) {
        mutations.forEach(function(mutation) {
            if (mutation.type === 'childList' && mutation.addedNodes.length > 0) {
                initializeAllProductGrids();
            }
        });
    });

    var config = { childList: true, subtree: true };
    observer.observe(document.body, config);
});

- Create asset ```component-card-variant-swatch-custom.css```
  ```css
    .card-product-custom-div {
      visibility: hidden;
  }
  
  .card-product-custom-div.loaded {
      visibility: visible;
  }
  
  .card__content.card-swatch__content,
  .card__information.card-swatch__information {
    padding-top: 0;
  }  .collection-product-card__swatch-variants {     position: relative;     z-index: 2;     display: flex;     flex-wrap: wrap;     align-items: center;     margin-bottom: 10px;   }

  .collection-product-card__swatch {
      display: inline-block;
      margin-top: 5px;
      margin-right: 5px;
  }
  
  .collection-product-card__swatch input {
      display: none;
  }
  
  .collection-product-card__swatch label {
      display: block;
      width: var(--card-swatch-size);
      height: var(--card-swatch-size);
      border: 1px solid #777;
      border-radius: var(--card-swatch-border-radius);
      background-size: cover;
      cursor: pointer;
      transition: border-color 0.3s ease;
  }
  
  .collection-product-card__swatch label:hover {
      border-color: #333;
  }
  
  .collection-product-card__swatch input:checked + label {
      border-color: #333;
      border-width: 2px; 
      box-shadow: inset 0 0 0 1px #fff; 
  }  
  
  .collection-product-card__swatch input.disabled + label {
      opacity: 0.5;
      position: relative; 
  }
  
  .collection-product-card__swatch input.disabled + label::after {
      content: '';
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      background: linear-gradient(to bottom right, transparent 45%, rgba(255, 0, 0, 0.6) 50%, transparent 55%);
      pointer-events: none;
  }

  .card-swatch--standard {
    padding: 0 0;
  }
  
  .card-swatch--card {
    padding: 0 2rem;
  }
  ```

  >Whatever we created from here is
    1. .liquid snippets for behaviour of swatch buttons
    2. .js file crtated for when click on swatch where we go
    3. .css file for swithces size color or images.

##  5. Edit Existing Theme Files
- Edit liquid snippet card-product.liquid
- Add code near top of file
  - You add these code  near line 30 for css
 ```liquid
{% assign use_card_swatches = false %}
{% if settings.card_swatches %}
  {% unless disable_card_swatches %}
    {% assign use_card_swatches = true %}
  {% endunless %}
{% endif %}
{% if use_card_swatches %} 
  <script>
  var variantDataMap{{ section.id | replace: '-', '_' }}_{{ card_product.id | replace: '-', '_' }} = {
    {% assign isFirstVariant = true %}  
      {% for variant in card_product.variants %}
        {% if variant.featured_media %}
          {% unless isFirstVariant %},{% endunless %}
          "{{ variant.id }}": {
            "imageUrl": "{{ variant.featured_media | img_url: 'master' }}",
            {% if card_product.url contains '?' %}
              "productUrl": "{{ card_product.url | append: '&variant=' | append: variant.id }}"
            {% else %}
              "productUrl": "{{ card_product.url | append: '?variant=' | append: variant.id }}"
            {% endif %}
          }
          {% assign isFirstVariant = false %} 
        {% endif %}
      {% endfor %}
  };
  </script>
  <style>
    :root {
      --card-swatch-size: {{ settings.card_swatch_size_custom }}px;
      --card-swatch-border-radius: {% if settings.card_swatch_shape_custom == 'circle' %}50%{% else %}0%{% endif %};
    }
  </style>
  {{ 'component-card-variant-swatch-custom.css' | asset_url | stylesheet_tag }}
  
  {% assign variant_option_name = settings.card_variant_option_name %}
  {% assign lazy_load_all_variants = settings.card_swatch_lazy_load %} 
  {% if card_product.metafields.custom.variant_swatch_map_override.value %}
    {% assign target_entry = card_product.metafields.custom.variant_swatch_map_override.value %}
  {% else %}  
    {% assign entry_title = settings.card_variant_swatch_metaobject %}
    {% assign target_entry = nil %}
    {% for entry in shop.metaobjects.variant_swatch_map.values %}
      {% if entry.title == entry_title  %}
        {% assign target_entry = entry %}
        {% break %}
      {% endif %}
    {% endfor %}
  {% endif %}

  {% if target_entry %}
    {% assign variant_images_data = target_entry.variant_images_json %}
  {% else %}
    {% assign variant_images_data = nil %}
  {% endif %}
  
  <span class="card-product-custom-div" 
    data-section-id="{{ section.id }}" 
    data-product-id="{{ card_product.id }}"
    data-lazy-load-all-variants="{{ lazy_load_all_variants }}"
  >
{% endif %}
```
- inside same folder search for ```Card__content```
  - We found two code with this name. Paste below first code.
```liquid
    {% if use_card_swatches %} 
      <div class ="collection-product-card__variants card-swatch--{{ settings.card_style }}">
        {% assign card_has_swatches = false %}
        {%- for card_product_option in card_product.options_with_values -%}
            {% if card_product_option.name == variant_option_name %}  
                {% assign card_has_swatches = true %}
                {% assign variant_options_images = variant_images_data.value | where: "variant_name", card_product_option.name %}       
                {% render 'card-variant-swatch-custom', product: card_product, option: card_product_option, variant_images_data: variant_options_images, lazy_load_all_variants: lazy_load_all_variants %}
            {% endif %}
        {%- endfor -%}
      </div>  
    {% endif %}
```
> Add conditional classes to the card content and information
replace this classes name div class="card__content ,class="card__information  to below code
```
      <div class="card__content {% if use_card_swatches and card_has_swatches %}card-swatch__content{% endif %}">
        <div class="card__information {% if use_card_swatches and card_has_swatches %}card-swatch__information{% endif %}">

```
> one more near line 659 we found liquid code start there before that code we need to close the classes.Paste this code there
```liquid
{% if use_card_swatches %} 
  </span>  
{% endif %}
```
### next we need theme.liquid file so open it
Edit **theme.liquid**
```liquid
{% if settings.card_swatches %} 
   <script src="{{ 'card-product-variant-selection-custom.js' | asset_url }}" defer="defer"></script>
{% endif %}
```

### Edit main-search.liquid and related-products.liquid
- Update both **main-search.liquid** and **related-products.liquid** code by adding **data-id="{{ section.id }}"** to the element with **class grid product-grid** that contains the card-product.liquid render
  ```liquid
    <ul
    data-id="{{ section.id }}"
    class="grid product-grid  grid--{{ section.settings.columns_mobile }}-col-tablet-down grid--{{ section.settings.columns_desktop }}-col-desktop"
    role="list"
  >
  ```

** Edit global.js**

Add **this.executeScripts(this);** to the loadRecommendations method of class ProductRecommendations
in if condition

Add method executeScripts(element) to the class ProductRecommendations
```js
  executeScripts(element) {
    const scripts = element.querySelectorAll('script');
    scripts.forEach((script) => {
      const newScript = document.createElement('script');
      newScript.textContent = script.textContent;
      newScript.async = false;
      document.body.appendChild(newScript).parentNode.removeChild(newScript);
    });
  }
```
