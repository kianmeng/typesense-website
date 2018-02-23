---
layout: page
title: Getting Started
nav_label: guide
permalink: /guide/
---

<div class="row no-gutters">
  <div id="doc-col" class="col-md-8">
    <p>Let's get you started with a quick guide that will show you how to install Typesense, index some documents
    and explore the data with some search queries.</p>

    <p>If you are looking for a detailed dive into Typesense, refer to our <a href="/api">API documentation</a>.</p>

    <h3 id="install-typesense">Installing Typesense</h3>

    <p>We have pre-built binaries available for Linux (X86_64) and Mac OS X from our
      <a href="/downloads">downloads page</a>.</p>

    <p>We also publish official Docker images for Typesense on
      <a href="https://hub.docker.com/r/typesense/typesense/">Docker hub</a>.</p>

    <h3 id="start-typesense">Starting the Typesense server</h3>

    <p>You can start Typesense with minimal options like this:</p>

    {% code_block run-binary %}
    ```shell
      mkdir /tmp/typesense-data
      ./typesense-server --data-dir=/tmp/typesense-data --api-key=$TYPESENSE_API_KEY
    ```
    {% endcode_block %}

    <p>On Docker, you can run Typesense like this:</p>

    {% code_block run-docker %}
    ```shell
      mkdir /tmp/typesense-data
      docker run -p 8108:8108 -v/tmp/typesense-data:/data typesense/typesense:0.8.0 \
        --data-dir /data --api-key=$TYPESENSE_API_KEY
    ```
    {% endcode_block %}

    <h4 id="typesense-arguments">Server arguments</h4>

    <table class="table table-striped">
      <tr>
        <th>Parameter</th>
        <th>Required</th>
        <th>Description</th>
      </tr>
      <tr>
        <td>data-dir</td>
        <td>true</td>
        <td>Path to the directory where data will be stored on disk.</td>
      </tr>
      <tr>
        <td>api-key</td>
        <td>true</td>
        <td>
          API key that allows all operations.
        </td>
      </tr>
      <tr>
        <td>search-only-api-key</td>
        <td>false</td>
        <td>
          API key that allows only searches. Use this to define a separate key for making requests directly from
          Javascript.
        </td>
      </tr>
      <tr>
        <td>listen-address</td>
        <td>false</td>
        <td>
          Address to which Typesense server binds. Default: <code>0.0.0.0</code>
        </td>
      </tr>
      <tr>
        <td>listen-port</td>
        <td>false</td>
        <td>
          Port on which Typesense server listens.. Default: <code>8108</code>
        </td>
      </tr>
      <tr>
        <td>master</td>
        <td>false</td>
        <td>
          Starts the server as a read-only replica by defining the master Typesense server's address in <br />
          <code>http(s)://&lt;master_address&gt;:&lt;master_port&gt;</code> format.
        </td>
      </tr>
      <tr>
        <td>ssl-certificate</td>
        <td>false</td>
        <td>
          Path to the SSL certificate file. You must also define <code>ssl-certificate-key</code> to enable HTTPS.
        </td>
      </tr>
      <tr>
        <td>ssl-certificate-key</td>
        <td>false</td>
        <td>
          Path to the SSL certificate key file. You must also define <code>ssl-certificate</code> to enable HTTPS.
        </td>
      </tr>
      <tr>
        <td>log-dir</td>
        <td>false</td>
        <td>
          By default, Typesense logs to stdout and stderr. To enable logging to a file, provide a path to a
          logging directory.
        </td>
      </tr>
    </table>


    <h3 id="install-client">Installing a client</h3>

    <p>At the moment, we have clients for Javascript, Python, and Ruby. </p>
    <p>We recommend that you use our API client if it's available for your language. It's also easy to
    interact with Typesense through its simple, RESTful HTTP API.</p>

    {% code_block install %}
    ```ruby
    gem install typesense
    ```

    ```python
    pip install typesense
    ```
    {% endcode_block %}

    <h3 id="sample-application">Tour</h3>

    <p>At this point, we are all set to start using Typesense. We will create a Typesense collection, index
      some documents in it and try searching for them.</p>

    <p>To follow along, download this small dataset that we've put together for this walk-through.</p>

    <h4 id="init-client">Initializing the client</h4>

    <p>Let's begin by configuring the Typesense client by pointing it to the Typesense master node.</p>

    <p>Be sure to use the same API key that you used to start the Typesense server earlier.</p>

    {% code_block init-client %}
    ```ruby
      require 'typesense'

      Typesense.configure do |config|
        config.master_node = {
          host:     'localhost',
          port:     8108,
          protocol: 'http',
          api_key:  'abcd'
        }
      end
    ```
    ```python
      import typesense

      typesense.master_node = typesense.Node(
        host='localhost',
        port=8108,
        protocol='http',
        api_key='<API_KEY>'
      )
    ```
    ```shell
      export TYPESENSE_API_KEY='<API_KEY>'
      export TYPESENSE_MASTER='http://localhost:8108'
    ```
    {% endcode_block %}

    <p>That's it - we're now ready to start interacting with the Typesense server.</p>

    <h4 id="create-collection">Creating a "books" collection</h4>

    <p>In Typesense, a collection is a group of related documents that is roughly equivalent to a table in a relational database.
      When we create a collection, we give it a name and describe the fields that will be indexed when a document is
      added to the collection.</p>

    {% code_block create-collection %}
    ```ruby
      require 'typesense'

      books_schema = {
        'name' => 'books',
        'fields' => [
          {'name' => 'title', 'type' => 'string' },
          {'name' => 'authors', 'type' => 'string[]' },
          {'name' => 'image_url', 'type' => 'string' },

          {'name' => 'publication_year', 'type' => 'int32' },
          {'name' => 'ratings_count', 'type' => 'int32' },
          {'name' => 'average_rating', 'type' => 'float' },

          {'name' => 'authors_facet', 'type' => 'string[]', 'facet' => true },
          {'name' => 'publication_year_facet', 'type' => 'string', 'facet' => true }
        ],
        'token_ranking_field' => 'ratings_count'
      }

      Typesense::Collections.create(books_schema)
    ```

    ```python
      import typesense

      books_schema = {
        'name': 'books',
        'fields': [
          {'name': 'title', 'type': 'string' },
          {'name': 'authors', 'type': 'string[]' },
          {'name': 'image_url', 'type': 'string' },

          {'name': 'publication_year', 'type': 'int32' },
          {'name': 'ratings_count', 'type': 'int32' },
          {'name': 'average_rating', 'type': 'float' },

          {'name': 'authors_facet', 'type': 'string[]', 'facet': True },
          {'name': 'publication_year_facet', 'type': 'string', 'facet': True },
        ],
        'token_ranking_field': 'ratings_count'
      }

      typesense.Collections.create(schema)
    ```

    ```shell
      curl "http://localhost:8108/collections" -X POST -H "Content-Type: application/json" \
            -H "X-TYPESENSE-API-KEY: ${TYPESENSE_API_KEY}" -d '{
              "name": "books",
              "fields": [
                {"name": "title", "type": "string" },
                {"name": "authors", "type": "string[]" },
                {"name": "image_url", "type": "string" },

                {"name": "publication_year", "type": "int32" },
                {"name": "ratings_count", "type": "int32" },
                {"name": "average_rating", "type": "float" },

                {"name": "authors_facet", "type": "string[]", "facet": true },
                {"name": "publication_year_facet", "type": "string", "facet": true }
              ],
              "token_ranking_field": "ratings_count"
            }'
    ```
    {% endcode_block %}

    <p>For each field, we define its <code>name</code>, <code>type</code> and whether it's a <code>facet</code> field.
    A facet field allows us to cluster the search results into categories and let us drill into each of those categories.
    We will be seeing faceted results in action at the end of this guide.</p>

    <h4 id="index-documents">Adding books to the collection</h4>

    <p>We're now ready to index some books into the collection we just created.</p>

    {% code_block index-documents %}
    ```ruby
      # TODO
    ```
    ```python
      import json
      import typesense

      with open('/tmp/books.jsonl') as infile:
        for json_line in infile:
          book_document = json.loads(json_line)
          typesense.Documents.create('books', book_document)
    ```
    ```shell
        #!/bin/bash
        input="/tmp/books.jsonl"
        while IFS= read -r line
        do
          curl "$TYPESENSE_MASTER/collections/books/documents" -X POST \
          -H "Content-Type: application/json" \
          -H "X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY" \
          -d "$line"
        done < "$input"
    ```
    {% endcode_block %}

    <h4 id="search-collection">Searching for books</h4>

    <p>We will start with a really simple search query - let's search for <code>harry potter</code> and ask Typesense
    to rank books that have more ratings higher in the results.</p>

    {% code_block search-collection-1 %}
    ```ruby
      search_parameters = {
        'q'         => 'harry potter',
        'query_by'  => 'title',
        'sort_by'   => 'ratings_count:desc'
      }

      Typesense::Documents.search('books', search_parameters)
    ```

    ```python
      search_parameters = {
        'q'         : 'harry',
        'query_by'  : 'title',
        'sort_by'   : 'ratings_count:desc'
      }

      typesense.Documents.search('books', search_parameters)
    ```

    ```shell
      curl -H "X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY" \
      "$TYPESENSE_MASTER/collections/books/documents/search\
      ?q=harry+potter&query_by=title&sort_by=ratings_count:desc"
    ```
    {% endcode_block %}

    <h5>Sample response</h5>

    {% code_block search-collection-1-response %}
      ```json
      {
        "facet_counts": [],
        "found": 62,
        "hits": [
          {
            "highlight": {
              "title": "<mark>Harry</mark> <mark>Potter</mark> and the Philosopher's Stone"
            },
            "document": {
              "authors": [
                "J.K. Rowling", "Mary GrandPré"
              ],
              "authors_facet": [
                "J.K. Rowling", "Mary GrandPré"
              ],
              "average_rating": 4.44,
              "id": "2",
              "image_url": "https://images.gr-assets.com/books/1474154022m/3.jpg",
              "publication_year": 1997,
              "publication_year_facet": "1997",
              "ratings_count": 4602479,
              "title": "Harry Potter and the Philosopher's Stone"
            }
          },
          ...
        ]
      }
      ```
    {% endcode_block %}

    <p>In addition to returning the matching documents, Typesense also highlights where the query terms appear
      in a document via the <code>highlight</code> property.</p>

    <p>Want to actually see newest <code>harry potter</code> books returned first? No problem, we can change the
      <code>sort_by</code> clause to <code>publication_year:desc</code>:</p>

    {% code_block search-collection-2 %}
    ```ruby
      search_parameters = {
        'q'         => 'harry potter',
        'query_by'  => 'title',
        'sort_by'   => 'publication_year:desc'
      }

      Typesense::Documents.search('books', search_parameters)
    ```

    ```python
      search_parameters = {
        'q'         : 'harry',
        'query_by'  : 'title',
        'sort_by'   : 'publication_year:desc'
      }

      typesense.Documents.search('books', search_parameters)
    ```

    ```shell
      curl -H "X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY" \
      "$TYPESENSE_MASTER/collections/books/documents/search\
      ?q=harry+potter&query_by=title&sort_by=publication_year:desc"
    ```
    {% endcode_block %}

    <h5>Sample response</h5>

    {% code_block search-collection-2-response %}
      ```json
      {
        "facet_counts": [],
        "found": 62,
        "hits": [
        {
          "highlight": {
            "title": "<mark>Harry</mark> <mark>Potter</mark> and the Cursed Child..."
          },
          "document": {
            "authors": [
              "John Tiffany", "Jack Thorne", "J.K. Rowling"
            ],
            "authors_facet": [
              "John Tiffany", "Jack Thorne", "J.K. Rowling"
            ],
            "average_rating": 3.75,
            "id": "279",
            "image_url": "https://images.gr-assets.com/books/1470082995m/29056083.jpg",
            "publication_year": 2016,
            "publication_year_facet": "2016",
            "ratings_count": 270603,
            "title": "Harry Potter and the Cursed Child, Parts One and Two"
          }
        },
        ...
        ]
      }
      ```
    {% endcode_block %}

    <p>Now, let's tweak our query to only fetch books that are published before the year <code>1998</code>.
       To do that, we just have to add a <code>filter_by</code> clause to our query:
    </p>

    {% code_block search-collection-3 %}
    ```ruby
      search_parameters = {
        'q'         => 'harry potter',
        'query_by'  => 'title',
        'filter_by' => 'publication_year:<1998',
        'sort_by'   => 'publication_year:desc'
      }

      Typesense::Documents.search('books', search_parameters)
    ```

    ```python
      search_parameters = {
        'q'         : 'harry',
        'query_by'  : 'title',
        'filter_by' : 'publication_year:<1998',
        'sort_by'   : 'publication_year:desc'
      }

      typesense.Documents.search('books', search_parameters)
    ```

    ```shell
      curl -H "X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY" \
      "$TYPESENSE_MASTER/collections/books/documents/search\
      ?q=harry+potter&query_by=title&sort_by=publication_year:desc\
      &filter_by=publication_year:<1998"
    ```
    {% endcode_block %}

    <h5>Sample response</h5>

    {% code_block search-collection-3-response %}
      ```json
      {
        "facet_counts": [],
        "found": 24,
        "hits": [
          {
            "highlight": {
              "title": "<mark>Harry</mark> <mark>Potter</mark> and the Philosopher's Stone"
            },
            "document": {
              "authors": [
                  "J.K. Rowling", "Mary GrandPré"
              ],
              "authors_facet": [
                  "J.K. Rowling", "Mary GrandPré"
              ],
              "average_rating": 4.44,
              "id": "2",
              "image_url": "https://images.gr-assets.com/books/1474154022m/3.jpg",
              "publication_year": 1997,
              "publication_year_facet": "1997",
              "ratings_count": 4602479,
              "title": "Harry Potter and the Philosopher's Stone"
            }
          },
          ...
        ]
      }
      ```
    {% endcode_block %}

    <p>Finally, let's see how Typesense handles typographic errors. Let's search for <code>experyment</code> - noticed the
    typo there? We will also facet the search results by the authors field to see how that works.</p>

    {% code_block search-collection-4 %}
    ```ruby
      search_parameters = {
        'q'         => 'experyment',
        'query_by'  => 'title',
        'facet_by'  => 'authors_facet',
        'sort_by'   => 'average_rating:desc'
      }

      Typesense::Documents.search('books', search_parameters)
    ```

    ```python
      search_parameters = {
        'q'         : 'harry',
        'query_by'  : 'title',
        'facet_by' : 'authors_facet',
        'sort_by'   : 'average_rating:desc'
      }

      typesense.Documents.search('books', search_parameters)
    ```

    ```shell
      curl -H "X-TYPESENSE-API-KEY: $TYPESENSE_API_KEY" \
      "$TYPESENSE_MASTER/collections/books/documents/search\
      ?q=harry+potter&query_by=title&sort_by=average_rating:desc\
      &facet_by=authors_facet"
    ```
    {% endcode_block %}

    <p>As we can see in the result below, Typesense handled the typographic error gracefully and fetched the results
      correctly. The <code>facet_by</code> clause also gives us a neat break-down of the number of books written
    by each author in the returned search results.</p>

    <h5>Sample response</h5>

    {% code_block search-collection-4-response %}
      ```json
      {
        "facet_counts": [
          {
            "field_name": "authors_facet",
            "counts": [
                {
                    "count": 2,
                    "value": " Käthe Mazur"
                },
                {
                    "count": 2,
                    "value": "Gretchen Rubin"
                },
                {
                    "count": 2,
                    "value": "James Patterson"
                },
                {
                    "count": 2,
                    "value": "Mahatma Gandhi"
                }
            ]
          }
        ],
        "found": 3,
        "hits": [
          {
            "_highlight": {
              "title": "The Angel <mark>Experiment</mark>"
            },
            "document": {
              "authors": [
                  "James Patterson"
              ],
              "authors_facet": [
                  "James Patterson"
              ],
              "average_rating": 4.08,
              "id": "569",
              "image_url": "https://images.gr-assets.com/books/1339277875m/13152.jpg",
              "publication_year": 2005,
              "publication_year_facet": "2005",
              "ratings_count": 172302,
              "title": "The Angel Experiment"
            }
          },
          ...
        ]
      }
      ```
    {% endcode_block %}

    <p>We've come to the end of our little tour. For a detailed dive into Typesense,
      refer to our <a href="/api">API documentation</a>.</p>

  </div>

  <div class="col-md-1 row no-gutters"></div>

  <div class="col-md-2 row no-gutters">
    <nav id="navbar-docs" class="position-fixed navbar navbar-light">
      <nav class="nav nav-pills flex-column">
        <a class="nav-link" href="#install-typesense">Installing Typesense</a>
        <a class="nav-link" href="#start-typesense">Starting Typesense</a>
        <a class="nav-link" href="#install-client">Install a client</a>
        <a class="nav-link" href="#sample-application">Tour</a>
        <nav class="nav nav-pills flex-column">
          <a class="nav-link ml-3 my-1" href="#init-client">Initializing the client</a>
          <a class="nav-link ml-3 my-1" href="#create-collection">Creating a books collection</a>
          <a class="nav-link ml-3 my-1" href="#index-documents">Adding some books</a>
          <a class="nav-link ml-3 my-1" href="#search-collection">Searching for books</a>
        </nav>
      </nav>
    </nav>
  </div>
</div>

<div class="row">
  <div class="col-md-8">

  </div>
</div>