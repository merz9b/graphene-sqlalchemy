Please read [UPGRADE-v2.0.md](https://github.com/graphql-python/graphene/blob/master/UPGRADE-v2.0.md)
to learn how to upgrade to Graphene `2.0`.

---

# ![Graphene Logo](http://graphene-python.org/favicon.png) Graphene-SQLAlchemy [![Build Status](https://travis-ci.org/graphql-python/graphene-sqlalchemy.svg?branch=master)](https://travis-ci.org/graphql-python/graphene-sqlalchemy) [![PyPI version](https://badge.fury.io/py/graphene-sqlalchemy.svg)](https://badge.fury.io/py/graphene-sqlalchemy) [![Coverage Status](https://coveralls.io/repos/graphql-python/graphene-sqlalchemy/badge.svg?branch=master&service=github)](https://coveralls.io/github/graphql-python/graphene-sqlalchemy?branch=master)


A [SQLAlchemy](http://www.sqlalchemy.org/) integration for [Graphene](http://graphene-python.org/).

## Installation

For instaling graphene, just run this command in your shell

```bash
pip install "graphene-sqlalchemy>=2.0"
```

## Examples

Here is a simple SQLAlchemy model:

```python
from sqlalchemy import Column, Integer, String

from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class UserModel(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    last_name = Column(String)
```

To create a GraphQL schema for it you simply have to write the following:

```python
import graphene
from graphene_sqlalchemy import SQLAlchemyObjectType

class User(SQLAlchemyObjectType):
    class Meta:
        model = UserModel
        # only return specified fields
        only_fields = ("name",)
        # exclude specified fields
        exclude_fields = ("last_name",)

class Query(graphene.ObjectType):
    users = graphene.List(User)

    def resolve_users(self, info):
        query = User.get_query(info)  # SQLAlchemy query
        return query.all()

schema = graphene.Schema(query=Query)
```

Then you can simply query the schema:

```python
query = '''
    query {
      users {
        name,
        lastName
      }
    }
'''
result = schema.execute(query, context_value={'session': db_session})
```

You may also subclass SQLAlchemyObjectType by providing `abstract = True` in
your subclasses Meta:
```python
from graphene_sqlalchemy import SQLAlchemyObjectType

class ActiveSQLAlchemyObjectType(SQLAlchemyObjectType):
    class Meta:
        abstract = True

    @classmethod
    def get_node(cls, info, id):
        return cls.get_query(info).filter(
            and_(cls._meta.model.deleted_at==None,
                 cls._meta.model.id==id)
            ).first()

class User(ActiveSQLAlchemyObjectType):
    class Meta:
        model = UserModel

class Query(graphene.ObjectType):
    users = graphene.List(User)

    def resolve_users(self, info):
        query = User.get_query(info)  # SQLAlchemy query
        return query.all()

schema = graphene.Schema(query=Query)
```

### Full Examples

To learn more check out the following [examples](examples/):

- [Flask SQLAlchemy example](examples/flask_sqlalchemy)
- [Nameko SQLAlchemy example](examples/nameko_sqlalchemy)

## Contributing

Set up our development dependencies:

```sh
pip install -e ".[dev]"
pre-commit install  
```

We use `tox` to test this library against different versions of `python` and `SQLAlchemy`.
While developping locally, it is usually fine to run the tests against the most recent versions:

```sh
tox -e py37  # Python 3.7, SQLAlchemy < 2.0
tox -e py37 -- -v -s  # Verbose output
tox -e py37 -- -k test_query  # Only test_query.py 
```

Our linters will run automatically when committing via git hooks but you can also run them manually:

```sh
tox -e pre-commit
```
