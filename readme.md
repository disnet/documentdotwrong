An minimal repro of an IE/Edge `document.write` bug.

In Edge (tested on 14 and 15), navigate to

- https://disnet.github.io/documentdotwrong/fails1.html
- https://disnet.github.io/documentdotwrong/fails2.html
- https://disnet.github.io/documentdotwrong/fails3.html
- https://disnet.github.io/documentdotwrong/works.html

Each page performs an XHR and then `document.write`'s the `responseText`. This should succeed (and does in Chrome/FF/Safari) but throws a `SCRIPT70: permission denied` error in Edge (and a similar `access is denied` error in IE 9-11).

The relevant failing code from `fails1.html`:

```js
addEventListener('load', function () {
  var xhr = new XMLHttpRequest();
  xhr.addEventListener('load', function() {
    document.open();
    document.write(xhr.responseText);
    document.close();
  });
  xhr.open("get", './success.html');
  xhr.send();
});
```

Ok, so the error says something about permissions, maybe Edge is getting confused about which `document` we're talking about so maybe be explicit in `fails2.html`:

```js
addEventListener('load', function () {
  var xhr = new XMLHttpRequest();
  xhr.addEventListener('load', function() {
    window.document.open();
    window.document.write(xhr.responseText);
    window.document.close();
  });
  xhr.open("get", './success.html');
  xhr.send();
});
```

Same error, so how about we store the `responseText` in a local variable (`fails3.html`):

```js
addEventListener('load', function () {
  var xhr = new XMLHttpRequest();
  xhr.addEventListener('load', function() {
    var text = xhr.responseText;
    document.open();
    document.write(text);
    document.close();
  });
  xhr.open("get", './success.html');
  xhr.send();
});
```

This page fails but only if dev tools is open so we're on the right track.

How about we combine the solution of `window.document` and the local variable (`works.html`):

```js
addEventListener('load', function () {
  var xhr = new XMLHttpRequest();
  xhr.addEventListener('load', function() {
    var text = xhr.responseText;
    window.document.open();
    window.document.write(text);
    window.document.close();
  });
  xhr.open("get", './success.html');
  xhr.send();
});
```

This works with and without dev tools open.

Seems like a bug.

