---
title: YAML Script with condition
layout: post
category: CI/CD
---

**YAML Script with condition**

In one of my projects I worked on a CI/CD setup in React native Expo, where I we have requirement to add condition inside the script, and it was kind of took me a while to add a condition in a inline command, so I decided to wrire a small blog on it, maybe that will helo someone.

Yes it's possible to add condition in YAML script, check the following options,

1. Use multiline statement:
```yaml
test_job:
  script: |
    if [[ "$FOO" == "bar" ]]; then
      echo "yup"
    fi
  variables:
    FOO: "bar"
```

2. Single like condition
```yaml
test_job:
  script:
    - if [[ "$FOO" == "bar" ]]; then echo "yup"; else echo "nope"; fi
  variables:
    FOO: "bar"
```

3. Mix of both
```yaml
test_job:
  script:
    - export FOO=bar
    - | 
      if [[ "$FOO" == "bar" ]]; then
       echo "yup"
      fi
    - echo "and so on"
```

