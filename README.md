# ai-product-hunter-evaluation

Evaluation method and prompts of [AI Product Hunter](https://ai-producthunt.com)

## evaluation target period of the day

- from: UTC 09:00:01 (02:00:01 PDT, 01:00:01 PST) of the day before
- to: UTC 09:00:00 (02:00:00 PDT, 01:00:00 PST) of the day

### example: ranking of 2024-07-01

- from: 2024-06-30 09:00:01 UTC
- to: 2024-07-01 09:00:00 UTC

## evaluation process

The evaluation of the products is executed through these 4 steps.

- step1: fetch posts(products) from Producthunt
- step2: collect information of the products
- step3: generate understanding of the products
- step4: generate product evaluation of the products

### step1: fetch posts(products) from Producthunt

fetch all posts(products) of the target period using [Producthunt API](https://api.producthunt.com/v2/docs)

```gql
query ($postedAfter: DateTime, $postedBefore: DateTime, $after: String) {
  posts(
    after: $after
    postedAfter: $postedAfter
    postedBefore: $postedBefore
    order: NEWEST
  ) {
    edges {
      node {
        id
        name
        tagline
        description
        createdAt
        website
        productLinks {
          type
          url
        }
        comments(order: NEWEST, first: 1) {
          edges {
            node {
              userId
              body
              createdAt
            }
          }
        }
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### step2: collect information of the products

scrape the website and productLinks

### step3: generate understanding of the products

generate understanding using GenAI

#### model of step3

- gpt-4o

#### messages(prompts) of step3

- role: system

```txt
You are a professional ProductHunt reviewer.
Your job is to understand and evaluate(score) the products launched to ProductHunt today.
You have to figure out the product is really something valuable, based on the fact that the majority of the products is nonsense.
```

- role: user

```txt
As a first step, you have to understand the product.
Here is the details.

-------------------------------------------------------------------------------------

## input json

### format
{
  "productName": string,
  "tagline": string,
  "description": string,
  "firstComment": string | null,
  "webSiteHtmlContents": string[]
}

### field description

#### productName
- name of the product

#### tagline
- short marketing sentence of the product

#### description
- description from the person who hunted the product
- note: usually makers self hunt their products

#### firstComment
- comment from the person who hunted the product

#### webSiteHtmlContents
- scraped website contents of the product

## output json

### format
{
  "whatIsThis": string,
  "levelOfCompletion": "just-idea" | "prototype" | "more-than-MVP",
  "business": string,
  "originality": string,
  "char50Description": string
}

### field description
#### whatIsThis
- describe the product with your word
- avoid use what the product says. You have to judge the words are trustworthy. Try to understand with an unbiased view.
- "hearsay style" is preferable
- less than 100 words

#### business
- determine whether it is for-profit or non-profit
- If it is for-profit, determine whether it is an advertising model, a one-time purchase model, or a subscription model
- If there is a price listed, please write it down.

#### originality
- if there is any similar product and it is famous, introduce its name and describe the difference from it
- if there is no similar product, write that it has complete originality

#### char50Description
- describe the product in 50 chars or less.

-------------------------------------------------------------------------------------
```

- role: user

```txt
{
  "productName": ${name fetched in step1},
  "tagline": ${tagline fetched in step1},
  "description": ${description fetched in step1},
  "firstComment": ${firstComment fetched in step1},
  "webSiteHtmlContents": ${webSiteHtmlContents scraped in step2}
}
```

### step4: generate product evaluation of the products

generate evaluation using GenAI

#### model of step4

- gpt-4o

#### messages(prompts) of step4

- role: system

```txt
You are a professional ProductHunt reviewer.
Your job is to understand and evaluate(score) the products launched to ProductHunt today.
You have to figure out the product is really something valuable, based on the fact that the majority of the products is nonsense.
```

- role: user

```txt
At the first step, you have done understanding of the product.
As a second step, you have to evaluate(score) the product.

Here is the details.

-------------------------------------------------------------------------------------

## input json

### format

{
  "productName": string,
  "whatIsThis": string,
  "levelOfCompletion": "just-idea" | "prototype" | "more-than-MVP",
  "business": string,
  "originality": string,
}

## output json

{
  "plusElements": { "item": string, "score": number, "reason": string }[]
  "minusElements": { "item": string, "score": number, "reason": string }[]
}

### plusElements
- technical innovation
- conceptual innovation
- problem solver
- entertainment Value
- business potential
- social impact
- other plus(additional)

### plusElements scoring guideline(not applied to minusElements)
- min: 0, max: 100
- 0: There is nothing to be evaluated in this context(50 percentile)
- 10: seems hopeless(65 percentile)
- 20: seems not promising(65 percentile)
- 30: seems a little bit promising(80 percentile)
- 40: seems promising enough(90 percentile)
- 50: seems very promising(98 percentile)
- 60: seems super promising(99 percentile)
- 61: ~ 100: super promising better than ever

### minusElements

- risks(fraud, legal, social problem, fake product, etc...) min: 0, max: 100
- other minus(additional) min: 0, max:100

-------------------------------------------------------------------------------------
```

- role: user

```txt
{
  "productName": ${name fetched in step1},
  "whatIsThis": ${whatIsThis generated in step3},
  "levelOfCompletion": ${levelOfCompletion generated in step3},
  "business": ${business generated in step3},
  "originality": ${business generated in step3},
}
```

## License

[MIT](./LICENSE)
