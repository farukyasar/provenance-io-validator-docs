---
description: Wait, that's an NFT?
---

# Data Mapping

NFT stands for “Non-Fungible Token” and is used in blockchain ecosystems to represent a unique (non-fungible) digital asset (token) whose ownership is registered and tracked on a blockchain.

Provenance NFTs are typically financial assets, such as loans or funds. However, the type of asset on Provenance is not restricted, allowing for innovation in the financial services industry. The [metadata-asset-model](https://github.com/provenance-io/metadata-asset-model) provides a highly extensible [Google Protocol Buffer](https://developers.google.com/protocol-buffers) representation of an [Asset](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/asset.md) to contain whatever data model the asset originator wishes to use.

{% hint style="info" %}
As you will see below, the Figure Tech team has put together a Loan Package data model that balances the needs of loan originators, validators, servicers, investors, and eNote controllers that we suggest you follow. The suggested data model remains highly extensible and allows mortgage originators to choose whether to map their own data model to the suggested layout, or simply onboard loan data in a MISMO XML format.
{% endhint %}

## Loan Package

{% hint style="info" %}
This section builds on the [Metadata Module](https://docs.provenance.io/modules/metadata-module) documentation. Revisit that page for definitions of the four core state objects in p8e: Contracts, Records, Sessions, and Scopes.
{% endhint %}

Asset originators must understand the structure of the scope they are going to onboard to Provenance Blockchain, also known as the Record Specification. We will stick to the mortgage as an example of an asset that we want to represent as an asset in Provenance. The Loan Package Record Specification is one scope that includes the following facts:

| Fact             | Description                                                                                                           | Nullable                                                 |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| Asset            | Core information about the loan that should not change throughout the life cycle of the loan after the loan is funded | No                                                       |
| Servicing Rights | Identifies the servicer and optional sub-servicer                                                                     | No                                                       |
| Documents        | List of loan documents (metadata with pointers to their location)                                                     | Yes                                                      |
| Loan States      | List of loan states from servicing                                                                                    | Yes                                                      |
| Validation       | List of validation requests and validation results                                                                    | Yes                                                      |
| eNote            | Metadata belonging to the authoritative copy of the eNote, as well as identifiers for the controller and custodian    | Yes (only used when eNote is to be registered with DART) |

The Asset fact is quite loosely defined, however, the intention is for loan originators to choose from one of two options:

1. Map their own data model to the [Loan Proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/loan/v1beta1/loan.proto#L29), which includes all of the fields needed to re-underwrite the loan and space to add custom fields in each section, or
2. Use the [MISMOLoan Proto](https://github.com/provenance-io/metadata-asset-model/blob/dkneisly/mismo-xml-as-loan/src/main/proto/tech/figure/loan/v1beta1/mismo\_loan.proto#L18), which simply requires a Universal Loan Identifier (ULI) and MISMO XML file, while allowing loan originators to extend the model as needed

{% hint style="info" %}
To help you choose, consider which format your business partners can handle. For example, if you intend to have your loan(s) validated by a third party that can handle version 3.4 of the MISMO Reference Model, then use the MISMOLoan Proto.

Both DART and Portfolio Manager can work off of either format.
{% endhint %}

See the [API Usage Guide](api-usage-guide.md) for a detailed, practical description of how each fact can get generated and memorialized throughout the loan application, closing, and validation processes. It is also not uncommon for loan originators to work with document preparation vendors that may integrate directly with Figure Tech to onboard and register eNotes with DART on closing day. A separate guide is available for onboarding eNotes separately from the rest of the loan package, to ensure direct transfer between eVaults.

### Proto Definitions

Each [Fact](../../../p8e/overview/#facts) in a [Scope](../../../p8e/overview/#scopes) is a key-value pair, where the key is a String and the value is a Protocol Buffer object. [Google Protocol Buffers](https://developers.google.com/protocol-buffers) support code generation in many languages. The table below lists the fact names and links to the Protocol Buffer definitions and documentation for each proto used in the loan scope.

| Fact Name        | Proto Definition                                                                                                                                                    | Proto Documentation                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| Asset            | [asset.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/asset/v1beta1/asset.proto#L19)                             | [asset.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/asset.md)           |
| Servicing Rights | [servicing\_rights.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/servicing/v1beta1/servicing\_rights.proto#L12) | [servicing.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/servicing.md)   |
| Documents        | [document.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/util/v1beta1/document.proto#L27)                        | [util.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/util.md)             |
| Loan States      | [loan\_state.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/servicing/v1beta1/loan\_state.proto#L23)             | [servicing.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/servicing.md)   |
| Validation       | [validation.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/tech/figure/validation/v1beta1/validation.proto#L13)              | [validation.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/validation.md) |
| eNote            | [registry.proto](https://github.com/provenance-io/metadata-asset-model/blob/main/src/main/proto/io/dartinc/registry/v1beta1/registry.proto#L14)                     | [registry.md](https://github.com/provenance-io/metadata-asset-model/blob/main/docs/regis.md)        |

### Examples

<details>

<summary>Example with Loan Proto</summary>

```kotlin
{
    "asset" : {
        "id": "c6978d46-3c3e-4175-a0d2-8f8ce47e8bb6",
        "type": "LOAN",
        "description": "MORTGAGE LOAN-1234",
        "kv": {
            "loan": {
                "typeUrl": "/tech.figure.asset.loan.Loan",
                "id": "c6978d46-3c3e-4175-a0d2-8f8ce47e8bb6",
                "originatorName": "Figure Lending",
                "originatorLoanId": "LOAN-1234",
                "borrowers": {
                    "primary": {
                        "partyType": "PRIMARY_BORROWER",
                        "name": {
                            "firstName": "FirstName",
                            "lastName": "LastName",
                            "middleName": "MiddleName",
                            "suffix": "NameSuffix"
                        }
                    }
                },
                "loanType": "MORTGAGE",
                "terms": {
                    "principalAmount": {
                        "amount": 1000000,
                        "currency": "USD"
                    },
                    "termInMonths": "360",
                    "rateType": "FIXED",
                    "interestRate": {
                        "value": "0.045"
                    },
                    "interestRateCap": {
                        "value": "0.045"
                    },
                    "payment": {
                        "firstPaymentAmount": {
                            "amount": 3000,
                            "currency": "USD"
                        },
                        "monthlyPaymentAmount": {
                            "amount": 3000,
                            "currency": "USD"
                        }
                    },
                    "dates": {
                        "initialOfferDate": {
                            "value": "2022-02-01"
                        },
                        "originationDate": {
                            "value": "2022-02-07"
                        },
                        "signedDate": {
                            "value": "2022-03-01"
                        },
                        "fundingDate": {
                            "value": "2022-03-06"
                        }
                    }
                },
                "funding": {
                    "status": "FUNDED",
                    "started": "2022-03-05T11:30:15.01Z",
                    "completed": "2022-03-06T04:30:15.01Z",
                    "disbursements": {
                        "id": {
                            "value": "<UUID>"
                        },
                        "amount": {
                            "amount": "1000000",
                            "currency": "USD"
                        },
                        "account": {
                            ...
                        },
                        "status": "COMPLETED",
                        "started": "2022-03-05T11:30:15.01Z",
                        "completed": "2022-03-06T04:30:15.01Z"
                    }
                },
                "mortgage": {
                    "lienProperty": {
                        "address": {
                            "street": "123 Main St",
                            "city": "City",
                            "state": "FL",
                            "zip": "33000"
                        }
                    },
                    "lienPosition": 1,
                    ...
                }
            }
        }
    },
    "servicingRights": {
        "servicerUuid": "<Servicer ID>",
        "servicerName": "Loan Servicing, Inc."
    },
    "documents": [
        {
            "id": "<UUID>",
            "uri": "EOS URI",
            "fileName": "Electronic Promissory Note (eNote)",
            "ContentType": "application/xml",
            "documentType": "MISMO_ENOTE_SMART_DOC_XML",
            "checksum": "<File sha512 Hash>"
        },
        ...
    ],
    "loanStates": null,
    "validation": null,
    "eNote": {
        "controllerId": {
            "controllerUuid": {
                "value": "<Controller ID>"
            },
            "controllerName": "<Controller Name>"
        },
        "eNote": {
            "id": "<UUID>",
            "uri": "EOS URI",
            "fileName": "Electronic Promissory Note (eNote)",
            "ContentType": "application/xml",
            "documentType": "MISMO_ENOTE_SMART_DOC_XML",
            "checksum": "<File sha512 Hash>"
        },
        "signedDate": {
            "value": "2022-03-01"
        },
        "vaultName": "DART eVault"
    }
}
```

</details>

<details>

<summary>Example with MISMOLoan Proto</summary>

```
{
    "asset" : {
        "id": "<uri>",
        "type": "MORTGAGE",
        "description": "MORTGAGE LOAN-1234",
        "kv": {
            "loan": {
                "typeUrl": "<mismo loan type>",
                "uri": "<uri>",
                "data": "<base64 encoded byte array>"
            }
        }
    },
    "servicingRights": {
        "servicerUuid": "<Servicer ID>",
        "servicerName": "Loan Servicing, Inc."
    },
    "documents": [
        {
            "id": "<UUID>",
            "uri": "EOS URI",
            "fileName": "Electronic Promissory Note (eNote)",
            "ContentType": "application/xml",
            "documentType": "MISMO_ENOTE_SMART_DOC_XML",
            "checksum": "<File sha512 Hash>"
        },
        ...remaining documents
    ],
    "loanStates": null,
    "validation": null,
    "eNote": {
        "controllerId": {
            "controllerUuid": {
                "value": "<Controller ID>"
            },
            "controllerName": "<Controller Name>"
        },
        "eNote": {
            "id": "<UUID>",
            "uri": "EOS URI",
            "fileName": "Electronic Promissory Note (eNote)",
            "ContentType": "application/xml",
            "documentType": "MISMO_ENOTE_SMART_DOC_XML",
            "checksum": "<sha512 Hash>"
        },
        "signedDate": {
            "value": "2022-03-01"
        },
        "vaultName": "DART eVault"
    }
}
```

</details>