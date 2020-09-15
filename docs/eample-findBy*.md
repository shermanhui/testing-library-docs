---
id: example-findBy*
title: example `findByText`
sidebar_label: findByText
---

```javascript
// src/__tests__/example.test.js
// This is an example of how to use findByText to query for text that
// is not visible right away

import {
  getByText,
  getByRole,
  findByText,
  getByPlaceholderText,
} from '@testing-library/dom'
import userEvent from '@testing-library/user-event'
// provides a set of custom jest matchers that you can use to extend jest
// i.e. `.toBeVisible`
import '@testing-library/jest-dom'

const renderContent = el => {
  el.innerHTML = `
  <form id="form_id" method="post" name="myform">
    <label>User Name<span id="username_required" style="color: red; display: none;"> required</span>:</label>
    <input type="text" name="username" id="username" placeholder="Enter user name"/>
    <span id="invalid_username" style="display: none;">Invalid User Name</span>
    <label>Password<span id="password_required" style="color: red; display: none;"> required</span>:</label>
    <input type="password" name="password" id="password" placeholder="Enter password"/>
    <span id="invalid_password" style="display: none;">Invalid Password</span>
    <input type="button" value="Login" id="submit" />
  </form>
  `

  const submitButton = el.querySelector('#submit')

  submitButton.addEventListener('click', () => {
    const userInput = document.getElementById('username')
    const passwordInput = document.getElementById('password')
    var userName = userInput.value
    var password = passwordInput.value

    if (!userName) {
      document.getElementById('username_required').style.display = 'inline'
    }

    if (!password) {
      document.getElementById('password_required').style.display = 'inline'
    }

    if (userName && userName !== 'Bob') {
      setTimeout(() => {
        document.getElementById('invalid_password').style.display = 'inline'
      }, 700)
    }

    if (password && password !== 'theBuilder') {
      setTimeout(() => {
        document.getElementById('invalid_password').style.display = 'inline'
      }, 700)
    }
  })

  return el
}

describe('findBy* Examples', () => {
  let div
  let container

  beforeEach(() => {
    div = document.createElement('div')
    container = renderContent(div)
  })

  it('should show a required field warning for each empty input field', async () => {
    expect(
      await findByText(container, (content, element) => {
        return content !== '' && element.textContent.startsWith('User Name')
      })
    ).toBeVisible()
    expect(
      await findByText(container, (content, element) => {
        return content !== '' && element.textContent.startsWith('Password')
      })
    ).toBeVisible()

    userEvent.click(
      getByRole(container, 'button', {
        name: 'Login',
      })
    )

    expect(
      await findByText(container, (content, element) => {
        return (
          content !== '' && element.textContent.includes('User Name required')
        )
      })
    ).toBeVisible()
    expect(
      await findByText(container, (content, element) => {
        return (
          content !== '' && element.textContent.includes('Password required')
        )
      })
    ).toBeVisible()
  })

  it('should show an async invalid password error', async () => {
    const userNameField = getByPlaceholderText(container, 'Enter user name')
    const passwordField = getByPlaceholderText(container, 'Enter password')

    userEvent.type(userNameField, 'wrongName')
    userEvent.type(passwordField, 'wrongPassword')
    // fireEvent.focus(passwordField);
    // fireEvent.change(passwordField, {
    //   target: {
    //     value: "wrong"
    //   }
    // });

    // fireEvent.click(
    //   getByRole(container, "button", {
    //     name: "Login"
    //   })
    // );
    userEvent.click(
      getByRole(container, 'button', {
        name: 'Login',
      })
    )
    // await waitFor(() => {
    //   expect(getByText(container, "Invalid Password")).toBeVisible()
    // })
    // screen.debug(await findByText(container, "Invalid Password"));
    expect(await findByText(container, 'Invalid User Name')).toBeVisible()
    expect(await findByText(container, 'Invalid Password')).toBeVisible()
  })
})
```
