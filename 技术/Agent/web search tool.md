

| 网站                                     |                        |
| -------------------------------------- | ---------------------- |
| tavily                                 |                        |
| exa                                    |                        |
| exa mcp                                | https://mcp.exa.ai/mcp |
| Searxng                                |                        |
| firecrawl.dev                          |                        |
| https://r.jina.ai/                     |                        |
| markdown.new                           |                        |
| scrapling extract get 'url' content.md | 微信文章                   |

```python

import time
from scrapling.fetchers import StealthyFetcher # pip install "scrapling[all]" && scrapling install

# 1. 定义一个交互函数，参数名必须接收 page 对象
def signup_action(page):    
    page.get_by_text('Sign up', exact=True).click(); time.sleep(3)

    # 使用 CSS 选择器定位并输入文字 (fill)
    page.get_by_role("textbox", name="Email address").fill("tavily04@zzftt.cloudns.biz"); time.sleep(3)
    frame = page.frame_locator('iframe'); print(frame)
    frame.locator('#success-text').wait_for(state='visible', timeout=30000)

    # 点击提交按钮 (click)
    page.get_by_role("button", name="Continue", exact=True).click(); time.sleep(3)

    # login
    if False:
        page.wait_for_selector('#password', timeout=30*1000)
        page.fill('#password', 'Tmpyb06011228++'); time.sleep(3)
        # page.press('#password', 'Enter'); time.sleep(3)
        page.get_by_role("button", name="action", exact=True).click(); time.sleep(3)
    else:
        code = 'xxxxxx'
        page.wait_for_selector('#code', timeout=30*1000)
        page.fill('#code', code); time.sleep(3)
        page.get_by_role("button", name="action", exact=True).click(); time.sleep(3)

    time.sleep(60)

# 2. 将交互函数通过 page_action 传给 Fetcher
# 当 Scrapling 打开网页后，会立刻执行 login_action，然后再捕获最终的网页源码
page_result = StealthyFetcher.fetch('https://app.tavily.com/home', headless=False, page_action=signup_action)  # 传入刚才定义的函数signup_action

```

```python

import time
from playwright.sync_api import sync_playwright  # pip install playwright && playwright install
from playwright_stealth import Stealth  # pip install playwright-stealth

with Stealth().use_sync(sync_playwright()) as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.goto("https://app.tavily.com/home")
    page.wait_for_timeout(10000)
    page.get_by_role("textbox", name="Email address").fill("tavily04@zzftt.cloudns.biz"); time.sleep(1)
    page.get_by_role("button", name="Continue", exact=True).click(); time.sleep(5)
    page.screenshot(path="stealth_screenshot.png")
    browser.close()

```

### openrouter注册

```python
def reg_action(page):
    import time
    print('click: Sign Up'); page.get_by_text('Sign Up', exact=True).nth(0).click()

    print('wait for: emailAddress'); page.wait_for_selector('#emailAddress-field', timeout=30*1000)
    print('input: emailAddress'); page.locator('#emailAddress-field').fill(email); time.sleep(3)

    print('wait for: password'); page.wait_for_selector('#password-field', timeout=30*1000)
    print('input: password'); page.locator('#password-field').fill(password); time.sleep(2)

    print('click: legalAccepted'); page.locator('#legalAccepted-field').click(); time.sleep(1)
    print('click: Continue'); page.get_by_text('Continue', exact=True).click(); time.sleep(1)
    
    try:
        print('wait for: iframe'); frame = page.frame_locator('iframe').first; print(frame)
        print('wait for: CF'); checkbox_label = frame.locator('.cb-lb').wait_for(timeout=15*1000); print('found .cb-lb')
        print('模拟鼠标移动'); page.mouse.move(700, 488); time.sleep(0.5);
        print('click: CF'); frame.locator('.cb-lb').click(); print('clicked'); time.sleep(10)
    except Exception as e:
        print(e)

    print('wait for: Verify email page'); page.wait_for_selector('.cl-headerTitle', timeout=300*1000); print('Verify email has send!')

def login_action(page):
    import time
    print('click: Sign Up'); page.get_by_text('Sign Up', exact=True).nth(0).click()

    print('wait for: emailAddress'); page.wait_for_selector('#emailAddress-field', timeout=30*1000)
    page.click('a:text-is("Sign in")');

    print('wait for: identifier'); page.wait_for_selector('#identifier-field', timeout=30*1000)
    print('input: identifier'); page.locator('#identifier-field').fill(email); time.sleep(3)
    
    print('wait for: password'); page.wait_for_selector('#password-field', timeout=30*1000)
    print('input: password'); page.locator('#password-field').fill(password); time.sleep(3)
    
    print('click: Continue'); page.get_by_text('Continue', exact=True).click(); time.sleep(3)
    
    otp_input = page.locator('input[data-input-otp="true"]')
    time.sleep(3)
    if otp_input.count() > 0:
        otp_input.fill(input("请输入验证码："));print("验证码已输入")
    else:
        print("OTP输入框不存在，跳过")
    
    page.wait_for_selector('button:has-text("Get API Key")', timeout=300*1000);
    page.click('button:has-text("Get API Key")');

    time.sleep(300);

    

proxy = 'http://username:password@31.59.20.176:6754'
email, password = 'ifu200jf@zzftt.cloudns.biz', '456rtyFGH++'

from scrapling.fetchers import stealthy_fetch
page = stealthy_fetch('https://openrouter.ai', headless=False, google_search=False, solve_cloudflare=True, page_action=reg_action, proxy=proxy) #; view(page)

```