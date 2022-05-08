---
title: Password Entropy Calculation (React Native)
layout: post
category: ReactNative
---

Today I will show you a modern way to of password validation, a Password Antropy Calculato. We will prepare an password entropy formula to calculate password strength, and we will prepare an animated password strength view to display password strength using different colors.

> Password strength is a measure of the effectiveness of a password against guessing or brute-force attacks. In its usual form, it estimates how many trials an attacker who does not have direct access to the password would need, on average, to guess it correctly. The strength of a password is a function of length, complexity, and unpredictability.

#### Why *Password Entropy*?
In this digital world *Password Entropy* signifies a **measure of password strength**, i.e., how effective a password is against adversaries who try to guess it or use a **brute-force attack**. A brute force attack means that someone sets up a script to try all possible combinations of characters to find the password. Such methods eventually would determine your password. So your only chance is to use a password that would take a **very long time to guess**.

> The number of trials an adversary would need to guess your password is an excellent measure of the password strength. This measure is known as **password entropy**.

#### The Formula

Here is Mathemetical formula to calculate *Password Entropy*:

<center><b>E = L * log(R) / log(2)</b></center>

where:

**R** - Size of the pool of unique characters from which we build the password

**L** - Password length, i.e., the number of characters in the password


So now as per the password entropy formula is increasing either '''L''' or '''R''' will strength the password. Hence, to have a stronger password, you must either expand the **pool of characters** or make the **password longer**. In particular, a longer password from a smaller pool can be as effective as a shorter yet more complex one!

> :warning: **Never rely solely on entropy to decide whether to use a particular password!, instead ues a combination of password entropy and password dictionary to detect if password is from leacked password dictionary, like [zxcvbn](https://github.com/dropbox/zxcvbn).

#### How do I create a strong password?

1. Make sure your password cannot be found in any dictionary of leaked passwords. Make it as unique as possible.
2. Increase the length of the password.
3. Enlarge the pool of symbols from which you take characters. Use lower case letters, upper case letters, digits, and special symbols.

When is a password secure?
A password is secure if it has at least 50 bits of entropy and does not appear in any list of leaked passwords.

Password calculation in theory
- Determind the length of the password 
- Calculate the size of the pool from which you've taken the charactors, let's say if you use lower case letters and digits the poll size is 26 + 10 = 36.

Enough of the theory, let's see how we can implement that,

```javascript
import React, { useState, useEffect } from 'react'
import { View, Animated, ViewPropTypes } from 'react-native'
import PropTypes from 'prop-types'

const
    regexArr = [/[a-z]/, /[A-Z]/, /[0-9]/, /[^A-Za-z0-9]/],
    sizeOfPool = [26, 26, 10, 33],
    stepsCount = 5,
    colors = ['#EA3939', '#EA3939', '#FF4D14', '#FF6C3E', '#FFB53E', '#44BB3A'],
    labels = ["Too short", "Too weak", "Weak", "Normal", "Strong", "Secure"]

const PassStrengthView = props => {
    const
        [passStat, setPassStat] = useState(labels[0]),
        [animateVal, setAnimateVal] = useState(new Animated.Value(0)),
        [animateColor, setAnimateColor] = useState(new Animated.Value(0)),
        [progress, setProgress] = useState(props.progress ?? 0),
        [width, setWidth] = useState(0)

    function calculateProgress() {
        let R = 1
        regexArr.forEach((rgx, index) => rgx.test(props.password) ? R += sizeOfPool[index] : null)
        let E = props.password.length * Math.log(R) / Math.log(2)

        const step = (props.entropy + props.entropy/stepsCount) / stepsCount;
        const progressRaw = Math.floor(E / step);
        const finalProgress = progressRaw <= stepsCount ? progressRaw : stepsCount;
        if (progress != finalProgress) {
            setProgress(finalProgress)
            props.onPasswordStrengthChange && props.onPasswordStrengthChange(finalProgress)
        }
        return finalProgress;
    }
    useEffect(() => {
        if (width <= 0) return;

        let progressValue = calculateProgress()

        Animated.spring(animateVal, {
            bounciness: 0,
            toValue: (width / stepsCount) * progressValue,
            useNativeDriver: false,
        }).start()
        let passPoint = 0

        setPassStat(labels[progressValue])
        Animated.timing(animateColor, {
            toValue: passPoint,
            duration: 300,
            useNativeDriver: false,
        }).start()

    }, [props.password, width])

    const interpolateColor = colors[progress]

    return (
        <View style={{ alignSelf: 'center', width: '100%' }} onLayout={(event) => {
            setWidth(event.nativeEvent.layout.width)
        }}>
            <View style={[styles.backBar, { backgroundColor: '#80808050' }]} />
            <Animated.View style={[styles.mainBar, { backgroundColor: interpolateColor, width: animateVal }]} />
            {
                props.showLabels ?
                    progress > 0 ?
                        <Animated.Text style={{ marginTop: 5, color: interpolateColor }}>{passStat}</Animated.Text>
                        : null
                    : null
            }
        </View>
    )
}

const styles = {
    backBar: {
        height: 5,
        borderRadius: 0,
    },
    mainBar: {
        position: 'absolute',
        height: 5,
        borderRadius: 0
    }
}

PassStrengthView.propTypes = {
    minLength: PropTypes.number,
    showLabels: PropTypes.bool,
    maxLength: PropTypes.number,
    entropy: PropTypes.number,
    labels: PropTypes.array,
    password: PropTypes.string.isRequired,
    backgroundColor: ViewPropTypes.style,
    fromColor: ViewPropTypes.style,
    toColor: ViewPropTypes.style,
}

PassStrengthView.defaultProps = {
    minLength: 8,
    maxLength: 15,
    entropy: 50,
    showLabels: true,
    backgroundColor: 'gray',
    fromColor: 'red',
    toColor: 'green',
}

export default PassStrengthView
```

Here regexArr is the pool of charactors, which is an array of regex of small leters, capital laters, numbers and everything else. And sizeOfPool is size of the pool. Steps count is number of steps you want to display in progress bar, colors and labels are the parameters you want to display in each steps.

#### Conclusion
Password entropy is a good modern way to show how much your password is secure and to encorage users to enter a strong and secure password, and we can get rid of the ugly password validation methods. Password entropy is just one aspect of deciding which type of password would be considered secure. It may happen that two passwords have the same entropy, and one of them is reasonably strong while the other is extremely weak. This is because of password dictionaries, which are lists of leaked passwords that are available online. So if you enter a password from password dictionaries, no matter how strong your password entropy is your password is still week, because it can be easily hacked. So never only depend on password entropy, instead use it in combination of password dictionaries.