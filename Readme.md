## What is this project for?
This is for everyone who is comfortable with Mysql but new to Elasticsearch. Going through this, one can get a flavor of how elasticsearch queries are written. And I generally use this as a quick reference when writing elasticsearch queries.

**Please note that all the code samples are in ruby.**

## Populating data
* Download the [accounts.json](data/accounts.json)
* Run the below command
```
curl -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary "@accounts.json"
```
Please refer to this [doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/_exploring_your_data.html) for more details on the sample data!

## Syntax of the Queries
Just to DRY the amount of text that is getting in here, all the code samples below have just the query variable information. If you want to actually run the queries in your ruby console, then you should be doing the following
```
require 'elasticsearch'
client = Elasticsearch::Client.new log: true
res = client.search index: 'bank', type: 'account', body: query
```


## Basic Queries
#### Get all users
```
query = {
  query: {match_all: {}}
}
```

### Search for a string
#### Get all users with firstname as 'Amber' 
```
query = {
  query: {
    match: {firstname: 'Amber'} 
  }
}
```

#### Get all users who have 'Amber' as either their firstname or lastname
```
query = {
  query: {
    bool: {
      should:[
        {match: {firstname: 'Amber'} },
        {match: {lastname: 'Amber'} }
      ]
    }
  }
}
```

#### Get all users whose firstname starts with am
```
query = {
  query: {
    regexp: {
      "firstname": "am.*"
    }
  }
}
```

#### Get all users whose firstname or lastname starts with be
```
query = {
  query: {
    bool: {
      should: [
        {regexp: {firstname: "be.*"}},
        {regexp: {lastname: "be.*"}}
      ]
    }
  }
}
```

### Select only few fields
Fetch only firstname and balance of the users
```
query = {
  query: {
    match_all: {}
  },
  _source: [:firstname, :balance]
}
```

#### Order
List all users sorted in the ascending order of their balance
```
query = {
  query: {
    match_all: {}
  },
  _source: [:firstname, :balance],
  sort: {
    balance: {order: :desc }
  }
}
```

List all users in the ascending order of states and descending order of balance
```
query = {
  query: {
    match_all: {},
  },
  _source: [:firstname, :balance, :state],
  sort: {
    state: {order: :asc},
    balance: {order: :desc}
  }
}
```

#### Sum
Total of all users' balances
```
query = {
  query: {
    match_all: {},
  },
  size: 0,
  aggs: {
    total_balance: {sum: {field: :balance}}
  }
}
```
Access the value from 
```
res['aggregations']['total_balance']['value']
```


#### Max
User with maxium balance
```
query = {
  query: {
    match_all: {}
  },
  size: 0,
  aggs: {
    max_balance: {max: {field: :balance}}
  }
}

```




#### Group By, Count
Total number of users available grouped by state and in their descending order
```
query = {
  query: {
    match_all: {},
  },
  size: 0,
  aggs: {
    group_by_state: {terms: {field: :state}}
  }
}
```

#### Nested Aggregations
Find the total of users' balance grouped by state
```
query = {
  query: {
    match_all: {},
  },
  size: 0,
  aggs: {
    group_by_state: {
      terms: {field: :state},
      aggs: {
        total_balance_in_state: {sum: {field: :balance}}
      }
    }
  }
}
```

Total of users' balance grouped by state at first level and grouped by cities at the second level
```
query = {
  query: {
    match_all: {},
  },
  size: 0,
  aggs: {
    group_by_state: {
      terms: {field: :state},
      aggs: {
        group_by_city: {
          terms: {field: :city},
          aggs: {
            total_balance_in_city: {sum: {field: :balance}}
          }
        }
      }
    }
  }
}
```
