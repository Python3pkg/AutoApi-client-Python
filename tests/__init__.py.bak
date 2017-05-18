# -*- coding: utf-8 -*-
import json
import mock
import unittest

from auto_api.app import app
from auto_api.auth import _admin_manager_client


KEYS_TO_FIX = {
    'json': 'data',
    'params': 'query_string'
}


def _fix_kwargs(verb):
    def _http_verb(*args, **kwargs):
        for old_key, new_key in KEYS_TO_FIX.iteritems():
            if old_key in kwargs:
                kwargs[new_key] = kwargs.pop(old_key)
        if 'data' in kwargs:
            kwargs['data'] = json.dumps(kwargs['data'])
            kwargs['content_type'] = 'application/json'
        response = verb(*args, **kwargs)
        response.json = lambda: json.loads(response.data)
        return response
    return _http_verb


class ClientWrapper(object):

    def __init__(self, client):
        self.client = client

    def __getattr__(self, name):
        attribute = getattr(self.client, name)
        if callable(attribute):
            return _fix_kwargs(attribute)
        return attribute

    def __getitem__(self, name):
        return getattr(self, name)


class FunctionalTests(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        super(FunctionalTests, cls).setUpClass()
        cls.client_mock = mock.patch('auto_api_client.requests', ClientWrapper(app.test_client()))
        cls.client_mock.start()
        cls.test_url = 'http://localhost:8686'

    @classmethod
    def tearDownClass(cls):
        cls.client_mock.stop()
        super(FunctionalTests, cls).tearDownClass()

    @staticmethod
    def clean(api, collection):
        with _admin_manager_client(app) as client:
            client[api][collection].drop()
