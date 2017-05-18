# -*- coding: utf-8 -*-
import requests

from .exceptions import AuthException, ResourceException


class Client(object):

    def __init__(self, url):
        self.url = url

    def __getattr__(self, name):
        setattr(self, name, Api(self, name))
        return self[name]

    def __getitem__(self, name):
        return getattr(self, name)


class AutoApiHttp(object):

    def url(self):
        url = self.parent.url
        return "%s/%s" % (url() if callable(url) else url, self.id)

    def headers(self):
        return self.parent.headers()

    def _http(self, fun, url=None, **kargs):
        return fun(url or self.url(), headers=self.headers(), **kargs)


class Api(AutoApiHttp):

    def __init__(self, parent, api_name):
        self.parent = parent
        self.id = api_name
        self.logged = False
        self._headers = {}
        self._collections = []

    def headers(self):
        return self._headers

    def login(self, email, password):
        response = self._http(
            requests.post,
            url="%s/login" % self.parent.url,
            json={'api': self.id, 'email': email, 'password': password}
        )
        self.logged = response.status_code == 200
        self._headers = {} if not self.logged else {
            'X-Email': response.headers.get('X-Email'),
            'X-Token': response.headers.get('X-Token')
        }
        return self.logged

    def logout(self):
        self._http(
            requests.post,
            url="%s/logout" % self.parent.url,
            json={'api': self.id}
        )
        self.logged = False
        self._headers = {}
        for key in self._collections:
            delattr(self, key)

    def __getattr__(self, name):
        if self.logged:
            setattr(self, name, Collection(self, name))
            self._collections.append(name)
            return self[name]
        raise AuthException("Api must be logged, use login method")

    def __getitem__(self, name):
        return getattr(self, name)


class Collection(AutoApiHttp):

    def __init__(self, parent, collection_name):
        self.parent = parent
        self.id = collection_name

    def get(self, params=None):
        response = self._http(requests.get, params=params)
        if response.status_code == 200:
            return response.json()

    def post(self, json=None):
        response = self._http(requests.post, json=json)
        if response.status_code == 201:
            return response.json()

    def __getattr__(self, resource_id):
        return Resource(self, resource_id)

    def __getitem__(self, resource_id):
        return getattr(self, resource_id)


class Resource(AutoApiHttp):

    def __init__(self, parent, resource_id):
        self.parent = parent
        self.id = resource_id
        response = self._http(requests.get)
        if response.status_code != 200:
            raise ResourceException("Not found resource")
        json = response.json()
        self._items = json.keys()
        for key in json:
            setattr(self, key, json[key])

    def delete(self):
        response = self._http(requests.delete)
        return response.status_code == 204

    def put(self, json=None):
        response = self._http(requests.put, json=json)
        if response.status_code == 204:
            for key in self._items:
                if key != 'id':
                    delattr(self, key)
            for key in json:
                if key != 'id':
                    setattr(self, key, json[key])
            self._items = json.keys()
        return response.status_code == 204

    def patch(self, json=None):
        response = self._http(requests.patch, json=json)
        if response.status_code == 204:
            for key in json:
                if key != 'id':
                    setattr(self, key, json[key])
        return response.status_code == 204

    def __getitem__(self, resource_id):
        return getattr(self, resource_id)
