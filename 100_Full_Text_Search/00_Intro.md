## 全文搜索

现在我们已经了解了搜索结构化数据的一些简单例子，是时候来探索_全文搜索_了：怎样从全文字段中找出最相关的文档。

下面是全文搜索的两个做重要的部分：

相关性：
根据查询条件将结果按照相关性排序的能力，相关性由检索词频率/反向文档频率(TF/IDF)，坐标接近度，模糊匹配，或其他的一些算法来计算。

分析：
将一段文本分解成明确，规范的表征的过程，用以创建和查询倒排索引

只要我们在讨论相关性或分析，我们就是在谈论查询（queries），而不是过滤（filters）。

### 词和全文文本的对比



While all queries perform some sort of relevance calculation, not all queries
have an analysis phase.((("full text search", "term-based versus")))((("term-based queries"))) Besides specialized queries like the `bool` or
`function_score` queries, which don't operate on text at all, textual queries can
be broken down into two families:

Term-based queries::
+
--

Queries like the `term` or `fuzzy` queries are low-level queries that have no
analysis phase.((("fuzzy queries"))) They operate on a single term. A `term` query for the term
`Foo` looks for that _exact term_ in the inverted index and calculates the
TF/IDF relevance `_score` for each document that contains the term.

It is important to remember that the `term` query looks in the inverted index
for the exact term only; it won't match any variants like `foo` or
`FOO`.  It doesn't matter how the term came to be in the index, just that it
is.  If you were to index `["Foo","Bar"]` into an exact value `not_analyzed`
field, or `Foo Bar` into an analyzed field with the `whitespace` analyzer,
both would result in having the two terms `Foo` and `Bar` in the inverted
index.

--

Full-text queries::
+
--

Queries like the `match` or `query_string` queries are high-level queries
that understand the mapping of a field:

*  If you use them to query a `date` or `integer` field, they will treat the
   query string as a date or integer, respectively.

*  If you query an exact value (`not_analyzed`) string field,((("not_analyzed string fields", "match or query-string queries on"))) they will treat
   the whole query string as a single term.

* But if you query a full-text (`analyzed`) field,((("analyzed fields", "match or query-string queries on"))) they will first pass the
  query string through the appropriate analyzer to produce the list of terms
  to be queried.

Once the query has assembled a list of terms, it executes the appropriate
low-level query for each of these terms, and then combines  their results to
produce the final relevance score for each document.

We will discuss this process in more detail in the following chapters.
--

You seldom need to use the term-based queries directly. Usually you want to
query full text, not individual terms, and this is easier to do with the
high-level full-text queries (which end up using term-based queries
internally).

[NOTE]
====
If you do find yourself wanting to use a query on an exact value
`not_analyzed` field, ((("exact values", "not_analyzed fields, querying")))think about whether you really want a query or a filter.

Single-term queries usually represent binary yes/no questions and are
almost always better expressed as a ((("filters", "single-term queries better expressed as")))filter, so that they can benefit from
<<filter-caching,filter caching>>:

[source,js]
--------------------------------------------------
GET /_search
{
    "query": {
        "filtered": {
            "filter": {
                "term": { "gender": "female" }
            }
        }
    }
}
--------------------------------------------------
====
