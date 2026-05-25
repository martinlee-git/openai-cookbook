# text_comparison_examples

- 원문 저장소: `openai/openai-cookbook`
- 미러 저장소: `martinlee-git/openai-cookbook`
- 원문 문서: https://github.com/openai/openai-cookbook/blob/main/articles/text_comparison_examples.md
- 미러 경로: `articles/text_comparison_examples.md`

## 한글 요약

텍스트 비교 예 OpenAI API 임베딩 엔드포인트를 사용하여 텍스트 간의 관련성 또는 유사성을 측정할 수 있습니다. 텍스트에 대한 GPT 3의 이해를 활용하여 이러한 임베딩은 비지도 학습 및 전이 학습 설정의 벤치마크에서 최고 수준의 결과를 달성했습니다. 임베딩은 의미 체계 검색, 권장 사항, 클러스터 분석, 거의 중복 감지 등에 사용할 수 있습니다. 자세한 내용은 OpenAI의 블로그 게시물 공지를 참조하세요. 텍스트 및 코드 임베딩 소개(2022년 1월) 새롭고 향상된 임베딩 모델(2022년 12월) 다른 임베딩 모델과의 비교는 MTEB(Massive Text Embedding Benchmark) 리더보드 의미 체계 검색을 참조하세요. 임베딩은 자체적으로 검색하거나 더 큰 시스템의 기능으로 검색에 사용할 수 있습니다. 검색에 임베딩을 사용하는 가장 간단한 방법은 다음과 같습니다. 검색 전(사전 계산): 텍스트 코퍼스를 토큰 제한보다 작은 청크로 분할합니다(텍스트 임베딩의 경우 8,191개 토큰 3소). 각 텍스트 청크 삽입 S

## 핵심 발췌

자체 데이터베이스 또는 Pinecone, Weaviate 또는 Qdrant와 같은 벡터 검색 공급자에서 해당 임베딩을 찢어냅니다. 검색 시(라이브 컴퓨팅): 검색 쿼리를 포함합니다. 데이터베이스에서 가장 가까운 임베딩을 찾습니다. 상위 결과를 반환합니다. 검색에 임베딩을 사용하는 방법의 예는 embeddings.ipynb를 사용한 의미 체계 텍스트 검색에 나와 있습니다. 고급 검색 시스템에서는 임베딩의 코사인 유사성을 검색 결과 순위 지정 시 여러 기능 중 하나로 사용할 수 있습니다. 질문 답변 GPT 3에서 신뢰할 수 있고 정직한 답변을 얻는 가장 좋은 방법은 정답을 찾을 수 있는 소스 문서를 제공하는 것입니다. 위의 의미 검색 절차를 사용하면 문서 모음에서 관련 정보를 저렴하게 검색한 다음 프롬프트를 통해 해당 정보를 GPT 3에 제공할 수 있습니다.

## 원문 내용

# Text comparison examples

The [OpenAI API embeddings endpoint](https://beta.openai.com/docs/guides/embeddings) can be used to measure relatedness or similarity between pieces of text.

By leveraging GPT-3's understanding of text, these embeddings [achieved state-of-the-art results](https://arxiv.org/abs/2201.10005) on benchmarks in unsupervised learning and transfer learning settings.

Embeddings can be used for semantic search, recommendations, cluster analysis, near-duplicate detection, and more.

For more information, read OpenAI's blog post announcements:

- [Introducing Text and Code Embeddings (Jan 2022)](https://openai.com/blog/introducing-text-and-code-embeddings/)
- [New and Improved Embedding Model (Dec 2022)](https://openai.com/blog/new-and-improved-embedding-model/)

For comparison with other embedding models, see [Massive Text Embedding Benchmark (MTEB) Leaderboard](https://huggingface.co/spaces/mteb/leaderboard)

## Semantic search

Embeddings can be used for search either by themselves or as a feature in a larger system.

The simplest way to use embeddings for search is as follows:

- Before the search (precompute):
  - Split your text corpus into chunks smaller than the token limit (8,191 tokens for `text-embedding-3-small`)
  - Embed each chunk of text
  - Store those embeddings in your own database or in a vector search provider like [Pinecone](https://www.pinecone.io), [Weaviate](https://weaviate.io) or [Qdrant](https://qdrant.tech)
- At the time of the search (live compute):
  - Embed the search query
  - Find the closest embeddings in your database
  - Return the top results

An example of how to use embeddings for search is shown in [Semantic_text_search_using_embeddings.ipynb](../examples/Semantic_text_search_using_embeddings.ipynb).

In more advanced search systems, the cosine similarity of embeddings can be used as one feature among many in ranking search results.

## Question answering

The best way to get reliably honest answers from GPT-3 is to give it source documents in which it can locate correct answers. Using the semantic search procedure above, you can cheaply search through a corpus of documents for relevant information and then give that information to GPT-3 via the prompt to answer a question. We demonstrate this in [Question_answering_using_embeddings.ipynb](../examples/Question_answering_using_embeddings.ipynb).

## Recommendations

Recommendations are quite similar to search, except that instead of a free-form text query, the inputs are items in a set.

An example of how to use embeddings for recommendations is shown in [Recommendation_using_embeddings.ipynb](../examples/Recommendation_using_embeddings.ipynb).

Similar to search, these cosine similarity scores can either be used on their own to rank items or as features in larger ranking algorithms.

## Customizing Embeddings

Although OpenAI's embedding model weights cannot be fine-tuned, you can nevertheless use training data to customize embeddings to your application.

In [Customizing_embeddings.ipynb](../examples/Customizing_embeddings.ipynb), we provide an example method for customizing your embeddings using training data. The idea of the method is to train a custom matrix to multiply embedding vectors by in order to get new customized embeddings. With good training data, this custom matrix will help emphasize the features relevant to your training labels. You can equivalently consider the matrix multiplication as (a) a modification of the embeddings or (b) a modification of the distance function used to measure the distances between embeddings.