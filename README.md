
The original version of app.js is:
```javascript
var port = process.env.PORT || 3000,
    http = require('http'),
    fs = require('fs'),
    html = fs.readFileSync('index.html');

var log = function(entry) {
    fs.appendFileSync('/tmp/sample-app.log', new Date().toISOString() + ' - ' + entry + '\n');
};

var server = http.createServer(function (req, res) {
    if (req.method === 'POST') {
        var body = '';

        req.on('data', function(chunk) {
            body += chunk;
        });

        req.on('end', function() {
            if (req.url === '/') {
                log('Received message: ' + body);
            } else if (req.url = '/scheduled') {      // <-------- Here might have a bug.
                log('Received task ' + req.headers['x-aws-sqsd-taskname'] + ' scheduled at ' + req.headers['x-aws-sqsd-scheduled-at']);
            }

            res.writeHead(200, 'OK', {'Content-Type': 'text/plain'});
            res.end();
        });
    } else {
        res.writeHead(200);
        res.write(html);
        res.end();
    }
});

// Listen on port 3000, IP defaults to 127.0.0.1
server.listen(port);

// Put a friendly message on the terminal
console.log('Server running at http://127.0.0.1:' + port + '/');
```

All GET HTTP request will response HTTP 200 with default HTML content. To generate a test environment that can response 404 when request path not found, this updated the else part as following:
```javascript
...
		} else {
			if(req.url === '/'){
				res.writeHead(200);
				res.write(html);
			} else {
				res.writeHead(404);
				res.write('Not Found');
			}
			res.end();
		}
...
```

You can launch server and verify by using `curl -I http://localhost:3000/any`. The response should look like:

```
↪ curl http://localhost:3000/any
Not Found

↪ curl -I http://localhost:3000/any
HTTP/1.1 404 Not Found
Date: Mon, 09 Jul 2018 06:01:49 GMT
Connection: keep-alive

↪ curl -I http://localhost:3000
HTTP/1.1 200 OK
Date: Mon, 09 Jul 2018 06:03:28 GMT
Connection: keep-alive
```