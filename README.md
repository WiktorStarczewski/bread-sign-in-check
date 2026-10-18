# Bread sign-in sheet check

A static test page for Bread's sign-in sheet, `window.bread.auth.open(url)` in Bread's in-app browser, on release builds: an iPhone, an Android phone and the desktop app. Plain HTML with inline scripts, no build step and no dependencies.

- Main page, for Bread's in-app browser: https://wiktorstarczewski.github.io/bread-sign-in-check/
- Second-origin page, for Bread's in-app browser: https://raw.githack.com/WiktorStarczewski/bread-sign-in-check/main/second-origin.html (GitHub Pages serves every repo of an account from one origin, so raw.githack.com serves this repo's `main` branch from a second one)
- Desktop page, for Bread's desktop app: https://wiktorstarczewski.github.io/bread-sign-in-check/desktop.html

Results show under each button and in the log at the bottom of the main page, with the time. Buttons on the main page are in bold; **Finish** and its variants are on the sign-in page that opens in the sheet. The pages never write to the console, so a device log line with a test handle in it came from somewhere else.

## Checklist

Google and Apple sign-in are checked separately.

**Each phone**

- [ ] Cancel: **Sign in**. iPhone: Cancel on iOS's "Bread Wants to Use ... to Sign In" prompt, then sign in again, Continue, and Cancel in the sheet. Android: close the tab with X or Back. Expected: `{"cancelled":true}` each time.
- [ ] A second open: **Sign in twice**. Expected: the second call rejects `AUTH_BUSY` at once, and the first opens the sheet.
- [ ] An iframe: **Sign in from an iframe**, and **Sign in from an iframe on another origin**. Expected: the frame has no `window.bread.auth`, its direct call to Bread's bridge rejects `AUTH_NOT_MAIN_FRAME`, and no sheet opens.
- [ ] A parked dApp: **Sign in after a delay**, then park the dApp before the delay ends. Expected: `AUTH_UNAVAILABLE` in the log when you come back, and no sheet over the wallet.
- [ ] Process death: **Sign in**, and with the sheet open, iPhone: swipe Bread away in the app switcher and open it again; Android: run `adb shell am kill com.miden.wallet`, then press **Finish** in the tab. Expected: Bread starts without crashing and nothing resolves; with the main page open again, **Sign in** opens a sheet, not `AUTH_BUSY`.
- [ ] No callback URL in the logs: **Sign in**, then **Finish**. Expected: `{ url }` with `origin` naming the main page and `login` set to the test handle the sign-in page showed. Search the device log for that handle (iPhone: the Console app on a Mac; Android: `adb logcat -d | grep <handle>`). Expected: no line contains it.

**Android only**

- [ ] A Custom Tab closed right after it opens: **Sign in**, then close the tab at once. Expected: `{"cancelled":true}`.
- [ ] A tab closed while its page is still loading: **Slow page**, then close the tab while it shows "Still loading". Expected: `{"cancelled":true}`.
- [ ] No Custom Tabs browser: disable every browser app, then **Sign in**. Expected: `AUTH_UNAVAILABLE` at once and no tab. With a browser enabled again, **Sign in** opens a tab.
- [ ] Chrome's picture-in-picture: **Sign in**, minimize the tab, restore it, then **Finish**. Note which came back: `{ url }`, or `{"cancelled":true}` already when the tab was minimized.
- [ ] The first Custom Tab after installing the browser: reinstall the browser or clear its data, **Sign in**, go through its first-run screens, then **Finish**. Expected: `{ url }`, not an early `{"cancelled":true}`.
- [ ] A default browser other than Chrome: make another browser with Custom Tabs the default, **Sign in**, then **Finish**. Expected: `{ url }`.
- [ ] After each Android check the page has an answer, and the next **Sign in** opens a tab, never `AUTH_BUSY`.

**Both phones**

- [ ] A page on a second origin: open the second-origin page, **Sign in**, then **Finish** in the sheet. Expected: `AUTH_ORIGIN_MISMATCH`, never `{ url }`.

**Desktop app**

- [ ] Open the main page. Expected status: `window.bread` present, no keys, frozen, no `version`, `presentation` or `auth`.
- [ ] Open the desktop page. Expected: `window.bread.own === true`, and a pass.

**Extra checks**

- [ ] In the sheet from **Sign in**: **Finish naming another origin**, **Finish with no origin** and **Finish with two origin parameters** each give `AUTH_ORIGIN_MISMATCH`; **Finish with login_error=cancelled** gives `{ url }` with `login_error=cancelled`.
- [ ] **Open a URL on another origin**: `AUTH_INVALID_URL`, and no sheet.

The slow page holds its load with an image from httpbin.org that answers after 25 seconds, plus a 10 second one from httpbingo.org in case httpbin.org is down, and counts the seconds it has been loading.
