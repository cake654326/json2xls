json2xls:Generate Excel by JSON data
====================================

[![](https://badge.fury.io/py/json2xls.png)](http://badge.fury.io/py/json2xls)
[![](https://pypip.in/d/json2xls/badge.png)](https://pypi.python.org/pypi/json2xls)

       _                 ____       _
      (_)___  ___  _ __ |___ \__  _| |___
      | / __|/ _ \| '_ \  __) \ \/ / / __|
      | \__ \ (_) | | | |/ __/ >  <| \__ \
     _/ |___/\___/|_| |_|_____/_/\_\_|___/
    |__/


**install**

    pip install json2xls

or

    python setup.py install

**generate excel by json string or json file**(each field in the json data is the same)

code:

    :::python
    from json2xls import Json2Xls

    json_data = '{"name": "ashin", "age": 16, "sex": "male"}'
    Json2Xls('test.xls', json_data).make()

command:

    python json2xls.py test.xls '{"a":"a", "b":"b"}'
    python json2xls.py test.xls '[{"a":"a", "b":"b"},{"a":1, "b":2}]'

    # from file: whole file is a complete json data
    python json2xls.py test.xls "`cat tests/data.json`"

    # from file: each line is a json data
    python json2xls.py test.xls tests/data2.json

excel:

    age | name | sex
    ----|------|----
    30  | John | male
    18  | Alice| female


**generate excel by request the url which return json data**

default request method is `get`, request argument pass by `params`.
and the `post` method's request argument pass by `data`, you can use `-d` to pass request data in command line, the data should be json or file


code:

    :::python
    from json2xls import Json2Xls

    url = 'http://api.bosonnlp.com/sentiment/analysis'
    Json2Xls('test.xlsx', url, method='post').make()

command:

    python json2xls.py test.xls http://api.bosonnlp.com/sentiment/analysis
    python json2xls.py test.xls http://api.bosonnlp.com/ner/analysis -m post -d '"我是傻逼"' -h "{'X-Token': 'bosontokenheader'}"

excel:

    status | message
    -------|--------
    403    | no token header

**Custom title and body**

default only support one layer json to generate the excel. if you want custom it,
you can write the `title_callback` function and `body_callback` function, the pass them in the `make` function.
for the `body_callback`, you just need to care one line data's write way.

example:

    :::python
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    import json
    import xlwt
    from json2xls import Json2Xls


    def title_callback(self, data):
        titles = [
                    u'title', u'text', u'forum', u'author.model',
                    u'author.verified_owner', u'linked',
                    u'linked.title', u'linked.forum'
                ]
        for col, name in enumerate(titles):
            try:
                width = self.sheet.col(col).width
                new_width = (len(name) + 1) * 256
                self.sheet.col(col).width = width if width > new_width else new_width
            except:
                pass
            self.sheet.row(0).write(col, name, self.title_style)
        self.start_row += 1

    def body_callback(self, data):
        values = []
        values.append(data['input']['meta']['title']['text'])
        values.append(data['input']['text'])
        values.append(data['input']['meta']['forum']['text'])
        values.append(data['input']['meta']['author']['model'])
        values.append(str(data['input']['meta']['author']['verified_owner']))
        values.append('\n'.join(json.dumps(x, ensure_ascii=False)
                                for x in data['output']['linked']))
        values.append('\n'.join(json.dumps(x, ensure_ascii=False)
                                for x in data['output']['meta']['title']['linked']))
        values.append('\n'.join(json.dumps(x, ensure_ascii=False)
                                for x in data['output']['meta']['forum']['linked']))

        for col, value in enumerate(values):
            try:
                self.sheet.row(self.start_row).height_mismatch = True
                self.sheet.row(self.start_row).height = 0
                width = self.sheet.col(col).width
                new_width = (len(value) + 1) * 256
                self.sheet.col(col).width = width if width > new_width else new_width
            except:
                pass
            self.sheet.row(self.start_row).write(col, value, xlwt.easyxf('align: wrap on;'))
        self.start_row += 1

    j = Json2Xls('tests/normalization.xls', 'tests/normalization.json')
    j.make(title_callback=title_callback, body_callback=body_callback)

