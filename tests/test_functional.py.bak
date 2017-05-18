# -*- coding: utf-8 -*-
from auto_api_client import Client, Collection
from auto_api_client.exceptions import AuthException, ResourceException
from tests import FunctionalTests


class TestAutoApi(FunctionalTests):

    def setUp(self):
        super(TestAutoApi, self).setUp()
        self.clean('api', 'collection')

    def test_login_and_logout(self):
        client = Client(self.test_url)
        with self.assertRaises(AuthException):
            client.api.collection
        self.assertTrue(client.api.login(email='admin', password='pass'))
        self.assertIsInstance(client.api.collection, Collection)
        client.api.logout()
        with self.assertRaises(AuthException):
            client.api.collection

    def test_empty_collection(self):
        client = Client(self.test_url)
        client.api.login(email='admin', password='pass')
        result = client.api.collection.get()
        client.api.logout()
        self.assertItemsEqual(result, [])

    def test_insert_and_retrieve_an_element(self):
        client = Client(self.test_url)
        client.api.login(email='admin', password='pass')
        item = client.api.collection.post(json={'key': 'value'})

        self.assertIn('id', item)
        result = client.api.collection.get()
        self.assertItemsEqual(result, [item])
        element = client.api.collection[item['id']]
        for key, value in item.iteritems():
            self.assertEqual(element[key], value)
        client.api.logout()

    def test_update_some_data(self):
        client = Client(self.test_url)
        client.api.login(email='admin', password='pass')
        item = client.api.collection.post(json={'key': 'value'})

        new_fields = {'key': 'other_value', 'other_key': 'eulav'}
        element = client.api.collection[item['id']]
        element.patch(json=new_fields)
        for key, value in new_fields.iteritems():
            self.assertEqual(element[key], value)

        element = client.api.collection[item['id']]
        for key, value in new_fields.iteritems():
            self.assertEqual(element[key], value)

        client.api.logout()

    def test_replace_all_data(self):
        client = Client(self.test_url)
        client.api.login(email='admin', password='pass')
        old_fields = {'key': 'value'}
        item = client.api.collection.post(json=old_fields)

        new_fields = {'value': 'key'}
        element = client.api.collection[item['id']]
        element.put(json=new_fields)
        for key in old_fields:
            if key not in new_fields:
                with self.assertRaises(AttributeError):
                    element[key]
        for key, value in new_fields.iteritems():
            self.assertEqual(element[key], value)

        element = client.api.collection[item['id']]
        for key in old_fields:
            if key not in new_fields:
                with self.assertRaises(AttributeError):
                    element[key]
        for key, value in new_fields.iteritems():
            self.assertEqual(element[key], value)

        client.api.logout()

    def test_insert_and_remove_an_element(self):
        client = Client(self.test_url)
        client.api.login(email='admin', password='pass')
        fields = {'key': 'value', 'value': 'key'}
        item = client.api.collection.post(json=fields)

        self.assertIn('id', item)
        element = client.api.collection[item['id']]
        self.assertTrue(element.delete())

        for key, value in fields.iteritems():
            self.assertEqual(element[key], value)

        with self.assertRaises(ResourceException):
            client.api.collection[item['id']]

        client.api.logout()
