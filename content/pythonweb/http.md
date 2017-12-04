---
title: "http"
date: 2017-12-04 18:37
---

# The httplib2.Http() objects are not thread-safe
```python
Traceback (most recent call last):
  File "/opt/cloud/meituan/client/mcclient/v1/shell/mebs.py", line 779, in run
    self.migrate(item)
  File "/opt/cloud/meituan/client/mcclient/v1/shell/mebs.py", line 760, in migrate
    zone = client.zones.get(mebs_template['zone_id'])
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 227, in get
    return self._get(url, self.keyword)
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 98, in _get
    resp, body = self.json_request('GET', url)
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 64, in json_request
    method, self._get_versioned_url(url), **kwargs)
  File "/opt/cloud/meituan/client/mcclient/v1/client.py", line 768, in json_request
    method, url, **kwargs)
  File "/opt/cloud/meituan/client/mcclient/v1/client.py", line 748, in _wrapped_request
    response = func(ep, t.get_token(), method, url, **kwargs)
  File "/opt/cloud/meituan/client/mcclient/common/http.py", line 129, in _json_request
    self.migrate(item)
  File "/opt/cloud/meituan/client/mcclient/v1/shell/mebs.py", line 760, in migrate
    zone = client.zones.get(mebs_template['zone_id'])
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 227, in get
    return self._get(url, self.keyword)
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 98, in _get
    resp, body = self.json_request('GET', url)
  File "/opt/cloud/meituan/client/mcclient/common/base.py", line 64, in json_request
    method, self._get_versioned_url(url), **kwargs)
  File "/opt/cloud/meituan/client/mcclient/v1/client.py", line 768, in json_request
    method, url, **kwargs)
  File "/opt/cloud/meituan/client/mcclient/v1/client.py", line 748, in _wrapped_request
    response = func(ep, t.get_token(), method, url, **kwargs)
  File "/opt/cloud/meituan/client/mcclient/common/http.py", line 129, in _json_request
    **kwargs)
  File "/opt/cloud/meituan/client/mcclient/common/http.py", line 114, in _http_request
    raise exceptions.from_response(resp, body)
BadRequest: Unable to communicate with API service: 'NoneType' object has no attribute 'makefile'. (HTTP 400): None
    **kwargs)
  File "/opt/cloud/meituan/client/mcclient/common/http.py", line 114, in _http_request
    raise exceptions.from_response(resp, body)
BadRequest: Unable to communicate with API service: 'NoneType' object has no attribute 'settimeout'. (HTTP 400): None
```

- [ The httplib2.Http() objects are not thread-safe](https://developers.google.com/api-client-library/python/guide/thread_safety)