# Embeddings Extension Specification

- **Title:** Embeddings
- **Identifier:** <https://jonas-hurst.github.io/embeddings/main/schema.json>
- **Field Name Prefix:** emb
- **Scope:** Item, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jonas-hurst

This document explains the Embeddings Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
Embeddings are an abstract vector representation of an input object. Recent advances in self-supervised learning has lead to a wide variety of
Foundation Models trained on earth observation imagery, from which general-purpose embeddings can be distilled.
These embeddings capture the essence of the input imagery.
These embeddings can then be used to train application-specific decoder heads (e.g. multi-layer perceptron).

This extension provides fields to STAC Collections and Items to enable discoverability of and cataloging of pre-computed embeddings.
The embeddings themselves are provided either as a downloadable asset (**recommended**) or as an item property (**discouraged**).
Providing embedding data for an item (as an Asset or as a property) is **requried**.

- Examples:
  - [Item example](examples/item_embedding_granular.json): Shows the usage of this extension when each item represents one embedding
  - [Item 2 example](examples/item_embedding_multiple.json): Shows the usage of this extension when each item can represent multiple embeddings
  - [Collection example](examples/collection.json): Shows the usage of this extension for collections that hold embeddings
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields for Collections and Items

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

They are general metadata on the embeddings provided. They **can** be specified on both the Collection and the Item level. To limit the volume
of data that needs to be transferred when querying STAC Items, it is however **recommended** to use them only on the Collection level that
holds the embedding itmes.

| Field Name                 | Type    | Description                                                                 |
| -------------------------- | ------- | --------------------------------------------------------------------------- |
| emb:model_id               | string  | ID of the model used to generate the embeddings                             |
| emb:model_name             | string  | Name of the model used to generate the embeddings                           |
| emb:model_version          | string  | Version of the model used to generate the embeddings                        |
| emb:model_family           | string  | Family of the model used to generate the embeddings                         |
| emb:model_description      | string  | Human-redable description of the model used to generate the embeddings      |
| emb:model_config           | string  | (Human-readable) configuration of the model used to generate the embeddings |
| emb:embedding_size         | integer | Size of the embedding vector                                                |
| emb:embedding_quantization | string  | Quantization method used for this vector                                    |
| emb:from_collection_id     | string  | ID of the collection from which the embeddings were generated               |
| emb:searchable             | boolean | Indicator whether search can be performed within the embedding space        |

### Additional Field Information

#### emb:model_config

Description of the model configuration needed to generate the embeddings, e.g. what layers were extracted

#### emb:embedding_size

The size of the embedding. A embedding vector with 750 entries would have this field set `emb:embedding_size: [750]`. In case of multi-dimensional embeddings, the n-th entry in the array represents the length of the embedding in the n-th dimension.

#### emb:embedding_quantization

If the embeddings are quantized, indicate the quantization method. If this field is not provided or the value is "None", no quantization is assumed

#### emb:from_collection_id

If the collection of images from which the embeddings were generated from resides in the same STAC as the embeddings, provided the collection ID
of the images using this field.

#### emb:searchable

Indicates, whether neighborhood search is possible within the latent embedding space. The (STAC) API for such an operation does not exist (yet),
therefore this field should not be used at the moment, or should be set to False.

## Fields for Items

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

They are specific to the embedding(s) provided within this STAC Item.

| Field Name             | Type           | Description                                                              |
| ---------------------- | -------------- | ------------------------------------------------------------------------ |
| emb:datetime_generated | string         | Date and time when the embeddings were generated                         |
| emb:from_item_id       | string         | ID of the ITEM from which the embeddings within this item were generated |
| emb:embedding          | array[number]  | **not recommended** The actual embedding                                 |
| emb:chip_pixels        | array[integer] | Index bounds that define the chip used to generate the embeddigns        |

### Additional Field Information

#### emb:from_item_id

If the image from which the embedding was generated, resides on this STAC as an Item within the collection given in `emb:from_collection_id`,
give the Item ID using this field.

#### emb:embedding

This field can only be used if one STAC Item equals represents one embedding. If one item represents multiple embedding
(e.g, all embeddings from one Senteinel-2 tile), this field can not be used to distribute the embeddings. In this case, distribute your embeddings
as downloadable assets to the Item.

This field can be used to transmit the embedding directly through the JSON document of the Item. This is **not recommended**, as it can increase the 
volume of data transmitted when querying STAC Items dramatically. Instead, provided the embeddings as an Item asset to be downloaded seperately.

#### emb:chip_pixels

This field contains information on the pixel indexes that form the chip (aka. patch, window) that was used to generate the embedding represented
by the Item. This fiel`can only be used if one STAC item represents one embedding.

This array of integers must be of length 4, the items within the array carry the following meaning: `[x_min, y_min, x_max, y_max]`.
For example, `[0, 0, 223, 223]` represents the upper left 224x224 pixels in the input image

## Asset objects

| (recommended) Field Name | Type         | role          | Description                                                           |
| ------------------------ | ------------ | ------------- | --------------------------------------------------------------------- |
| emb:embeddings           | Asset Object | emb:embeddings | **RECOMMENDED** Asset containing the embeddings described by the Item |

For easy discoverability, it is recommended that the key to the Asset object, that holds the embeddings corresponding to this item, is named `emb:embeddings`.
The `role` for this asset holding the embedding data **must** be `emb:embedding`, additionally the role `data` may be used.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type           | Description                                                                            |
| -------------- | -------------------------------------------------------------------------------------- |
| generated-from | Link to the collection or item, from which the embeddings were generated               |
| encoder-model  | Link to the model that was used to generate the embeddings                             |
| decoder-model  | Link to the model that can be used to reconstruct the origial data from the embeddings |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
