---
title: "Test Cases Like How We Speak"
date: 2019-08-06T09:12:13+08:00
tags: [ruby,agile development,unit test,tdd,test driven development]
categories: [engineering]
---

In my current software development work, we've talked about [TDD](https://en.wikipedia.org/wiki/Test-driven_development) within the team but we haven't really apply this process for couple reasons. But we do care about the products we shipped. So we all committed to the principle which is we should always write tests for the production code.

## Typical cases

Writing test cases is a way to communicate with your current and the future team members. Even more, it's a way to communicate with PMs or POs that the engineering artifact can satisfy the requirements and scenarios from stakeholders. Therefore, making test cases readable is important.

Test case always cover the following parts, inputs, testing condition, function execution and result verification. The following snippet is a typical test cases we have. There's a bunch of similar codes. Even more, they are just duplicates.

```ruby
class BudgetServiceTest
  def setup
    super
    @budget_repo = BudgetRepo.new
    @budget_service = BudgetService.new(@budget_repo)
  end

  test 'no budget' do
    stub(@budget_repo).all_budgets { [] }
    start_date = Date.strptime('20190401', '%Y%m%d')
    end_date = Date.strptime('20190401', '%Y%m%d')
    result = @budget_service.query(start_date, end_date)
    assert_equal 0, result
  end

  test 'period not in budget month' do
    budgets = [Budget.new('201904', 30), Budget.new('201905', 310)]
    stub(@budget_repo).all_budgets { budgets }
    start_date = Date.strptime('20190301', '%Y%m%d')
    end_date = Date.strptime('20190301', '%Y%m%d')
    result = @budget_service.query(start_date, end_date)
    assert_equal 0, result
  end

  test 'period inside budget month' do
    budgets = [Budget.new('201904', 30), Budget.new('201905', 310)]
    stub(@budget_repo).all_budgets { budgets }
    start_date = Date.strptime('20190401', '%Y%m%d')
    end_date = Date.strptime('20190401', '%Y%m%d')
    result = @budget_service.query(start_date, end_date)
    assert_equal 1, result
  end
end
```

To make this chunks more structured, we usually move some of them into a private methods. For example, extract data preparation to another method and extract function execution to another method. Each test cases then can be more clear.

```ruby
  ...

  test 'period inside budget month' do
    prepare_budgets
    result = query_result('20190401', '20190401')
    assert_equal 1, result
  end

  private

  def prepare_budgets
    budgets = [Budget.new('201904', 30), Budget.new('201905', 310)]
    stub(@budget_repo).all_budgets { budgets }
  end

  def query_result(start_date_str, end_date_str)
    start_date = Date.strptime(start_date_str, '%Y%m%d')
    end_date = Date.strptime(end_date_str, '%Y%m%d')
    @budget_service.query(start_date, end_date)
  end

  ...
```

Actually, this is already fine but maybe we would like to have more flexibility on what data is prepared or what query condition is defined. Further, those thing were under a certain scope.

## More descriptive by leveraging on language feature

There's a testing framework called [`rspec`](https://rspec.info/) which provides this flexibility. However, what if the existing code base does not use it? In Ruby, it's actually pretty simple to achieve almost the same thing by use fundamental elements, block and its invocation. Here's updated example.

```ruby
  ...

  test 'period inside budget month' do
    given_budgets Budget.new('201904', 30), Budget.new('201905', 310) do
      when_period from('20190401'), to('20190401') do
        result_should_be 1
      end
    end
  end

  private

  def given_budgets(*budgets)
    if block_given?
      stub(@budget_repo).all_budgets { budgets }
      yield
    end
  end

  def with_period(from, to)
    if block_given?
      assertion_block = yield
      actual = @budget_service.query(from, to)
      assertion_block.call(actual)
    end
  end

  def result_should_be(expected)
    -> (actual) { assert_equal expected, actual }
  end

  def query_result(from, to)
    @budget_service.query(from, to)
  end

  def from(date_str)
    Date.strptime(date_str, '%Y%m%d')
  end

  def to(date_str)
    Date.strptime(date_str, '%Y%m%d')
  end

  ...
```

## References

[Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development)

[Test Case](https://en.wikipedia.org/wiki/Test_case)