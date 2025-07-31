# POP-GOES-THE-WEASEL – 150 pts
Category: WEB

Solver: Cozy

Objective:
Find the flag.
## Challenge Overview
We were given a site that contains the flag but we don't know where.

My first approach to finding the flag was to inspect the site and crawl through its source.

```html
<!DOCTYPE html>
<html>
<head>
    <title>please dont break me</title>
</head>
<body>
    <h1>please dont break me</h1>
    <img src="https://upload.wikimedia.org/wikipedia/en/a/a4/Hide_the_Pain_Harold_%28Andr%C3%A1s_Arat%C3%B3%29.jpg" alt="Hide the Pain Harold">
    <script>
        const flagArray = [43,49,54,55,"7b",65,61,73,79,"5f","6c",61,"6d",61,"6e",67,"5f",76,69,65,77,73,"6f",75,72,63,65,"7d"];
        const flag = flagArray.map(charCode => String.fromCharCode(charCode)).join('');
        const urlParams = new URLSearchParams(window.location.search);
        const name = urlParams.get('name');
        if (name === 'CITU') {
            alert(flag);
        } else {
            throw new Error(flag);
        }
    </script>
</body>
</html>
```

```javascript
 const flagArray = [43,49,54,55,"7b",65,61,73,79,"5f","6c",61,"6d",61,"6e",67,"5f",76,69,65,77,73,"6f",75,72,63,65,"7d"];
```
And just like that!, I found the flag. But it looks like i have to convert this list of hex bytes and just need to turn them into ASCII.

The list spells out

```python
CITU{easy_lamang_viewsource}
```

