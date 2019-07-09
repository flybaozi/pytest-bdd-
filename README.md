## 语法讲解
| 关键字| 解释|
| ---  | ---|
| When |操作步骤 |
| Then |验证结果/数据清理 |
| Given|一般用来做预制条件  |
| And  |并且(用来配合When\Then\Given使用) |

```gherkin
Feature: Blog # 需求名称
    A site where you can publish your articles.
    # 需求行为描述

Scenario: Publishing the article
# 执行场景
    Given I'm an author user
    # 登录一个作者用户
    And I have an article
    # 我有一篇文章
    When I go to the article page
    # (当我)打开文章页面
    And I press the publish button
    # 然后我按下发布按钮
    Then I should not see the error message
    # 我应该看不到错误信息
    And the article should be published 
    # 然后文章应该发布成功
```

* * *

### scenario 装饰函数介绍

```python
@scenario
def test_xx():
    pass
```
装饰的函数
在所有场景步骤之后执行。可调用fixtures/或者做一些断言

**for example:**
```python
from pytest_bdd import scenario, given, when, then

@scenario('publish_article.feature', 'Publishing the article')
def test_publish(browser):
    assert article.title in browser.html
```


* * *
### 给测试用例定义多个行为名称
每个行为名称为独立，相互无关联

```python
@given('I have an article')
@given('there\'s an article')
def article(author):
    return create_test_article(author=author)
```

feature files example:
```gherkin
Scenario: I'm the author
    Given I'm an author
    And I have an article


Scenario: I'm the admin
    Given I'm the admin
    And there's an article
```



* * *
### scope argument参数
```python
@given('there is an article', scope='session')
def article(author):
    return create_test_article(author=author)
```
feature files example:
```gherkin
Scenario: I'm the author
    Given I'm an author
    And there is an article

Scenario: I'm the admin
    Given I'm the admin
    And there is an article
```

there is an article行为将被执行一次 即使多个场景复用它
顺序如下

1. I'm an author
2. there is an article
3. I'm the admin

* * *

### arguments传递
#### parsers.cfparse
```python
from pytest_bdd import parsers

@given(parsers.cfparse('there are {start:Number} cucumbers', extra_types=dict(Number=int)))
def start_cucumbers(start):
    return dict(start=start, eat=0)
```
```python
extra_types=dict(Number=int)
# 用来进行对于arguments参数的格式转化
```

#### parsers.re 
```python
from pytest_bdd import parsers

@given(parsers.re(r'there are (?P<start>\d+) cucumbers'), converters=dict(start=int))
def start_cucumbers(start):
    return dict(start=start, eat=0)
```
```python
converters=dict(start=int)
# 用来进行对于arguments参数的格式转化
```

feature files example:
```gherkin
Scenario: Arguments for given, when, thens
    Given there are 5 cucumbers

    When I eat 3 cucumbers
    And I eat 2 cucumbers

    Then I should have 0 cucumbers
```

* * *
### Override fixtures
```python
from pytest_bdd import given

@pytest.fixture
def foo():
    return "foo"


@given("I have injecting given", target_fixture="foo")
def injecting_given():
    return "injected foo"


@then('foo should be "injected foo"')
def foo_is_foo(foo):
    assert foo == 'injected foo'
```
理解如下：
原
```
foo()
```
被覆盖为
```
injecting_given() 
```
相当于 
原
```
return ”foo“ 
```
变为
```
return ”injected foo'“
```
