# The RSpec Style Guide

This RSpec style guide outlines the recommended best practices for real-world
programmers to write code that can be maintained by other real-world
programmers.

## How to Read This Guide

The guide is separated into sections based on the different pieces of an entire
spec file. There was an attempt to omit all obvious information, if anything is
unclear, feel free to open an issue asking for further clarity.

## A Living Document

Per the comment above, this guide is a work in progress - some rules are simply
lacking thorough examples, but some things in the RSpec world change week by
week or month by month. With that said, as the standard changes this guide is
meant to be able to change with it.

## Table of Contents

* [Layout](#layout)
* [Example Structure](#example-structure)
* [Naming](#naming)
* [Matchers](#matchers)
* [Rails](#rails)
  * [Views](#views)
  * [Controllers](#controllers)
  * [Models](#models)
  * [Mailers](#mailers)

## Layout

* <a name="empty-lines-after-describe"></a>
  Do not leave empty lines after `feature`, `context` or `describe`
  descriptions. It does makes the code more difficult to read and lowers the
  value of logical chunks.
<sup>[[link](#empty-lines-after-describe)]</sup>

  ```ruby
  # bad
  describe Article do

    describe '#summary' do

      context 'when there is a summary' do

        it 'returns the summary' do
          # ...
        end
      end
    end
  end

  # good
  describe Article do
    describe '#summary' do
      context 'when there is a summary' do
        it 'returns the summary' do
          # ...
        end
      end
    end
  end
  ```

* <a name="empty-lines-after-let"></a>
  Leave one empty line after `let`, `subject`, and `before`/`after` blocks.
<sup>[[link](#empty-lines-after-let)]</sup>

  ```ruby
  # bad
  describe Article do
    subject { FactoryBot.create(:some_article) }
    describe '#summary' do
      # ...
    end
  end

  # good
  describe Article do
    subject { FactoryBot.create(:some_article) }

    describe '#summary' do
      # ...
    end
  end
  ```

* <a name="let-grouping"></a>
  Only group `let`, `subject` blocks and separate them from `before`/`after`
  blocks. It makes the code much more readable.
<sup>[[link](#let-grouping)]</sup>

  ```ruby
  # bad
  describe Article do
    subject { FactoryBot.create(:some_article) }
    let(:user) { FactoryBot.create(:user) }
    before do
      # ...
    end
    after do
      # ...
    end
    describe '#summary' do
      # ...
    end
  end

  # good
  describe Article do
    subject { FactoryBot.create(:some_article) }
    let(:user) { FactoryBot.create(:user) }

    before do
      # ...
    end

    after do
      # ...
    end

    describe '#summary' do
      # ...
    end
  end
  ```

* <a name="empty-lines-around-it"></a>
  Leave one empty line around `it`/`specify` blocks. This helps to separate the
  expectations from their conditional logic (contexts for instance).
<sup>[[link](#empty-lines-around-it)]</sup>

  ```ruby
  # bad
  describe '#summary' do
    let(:item) { double('something') }

    it 'returns the summary' do
      # ...
    end
    it 'does something else' do
      # ...
    end
    it 'does another thing' do
      # ...
    end
  end

  # good
  describe '#summary' do
    let(:item) { double('something') }

    it 'returns the summary' do
      # ...
    end

    it 'does something else' do
      # ...
    end

    it 'does another thing' do
      # ...
    end
  end
  ```

* <a name="leading-subject"></a>
  When `subject` is used, it should be the first declaration in the example group.
<sup>[[link](#leading-subject)]</sup>

  ```ruby
  # bad
  describe Article do
    before do
      # ...
    end

    let(:user) { FactoryBot.create(:user) }
    subject { FactoryBot.create(:some_article) }

    describe '#summary' do
      # ...
    end
  end

  # good
  describe Article do
    subject { FactoryBot.create(:some_article) }
    let(:user) { FactoryBot.create(:user) }

    before do
      # ...
    end

    describe '#summary' do
      # ...
    end
  end
  ```

* <a name="redundant-before-each"></a>
  Don't specify `(:each)` for `before`/`after` blocks, as it is
  the default functionality. There are almost zero cases to use `before(:all)`
  anymore, but if you do find one - just write it out like `before(:all)`
<sup>[[link](#redundant-before-each)]</sup>

  ```ruby
  # bad
  describe '#summary' do
    before(:each) do
      subject.summary = 'something'
    end
  end

  # good
  describe '#summary' do
    before do
      subject.summary = 'something'
    end
  end
  ```

* <a name="should-in-it"></a>
  Do not write 'should' or 'should not' in the beginning of your `it` blocks.
  The descriptions represent actual functionality - not what might be happening.
<sup>[[link](#should-in-it)]</sup>

  ```ruby
  # bad
  it 'should return the summary' do
    # ...
  end

  # good
  it 'returns the summary' do
    # ...
  end
  ```

## Example Structure

* <a name="one-expectation"></a>
  Use only one expectation per example. There are very few scenarios where two
  or more expectations in a single `it` block should be used. So, general rule
  of thumb is one expectation per `it` block.
<sup>[[link](#one-expectation)]</sup>

  ```ruby
  # bad
  describe ArticlesController do
    #...

    describe 'GET new' do
      it 'assigns new article and renders the new article template' do
        get :new
        expect(assigns[:article]).to be_a(Article)
        expect(response).to render_template :new
      end
    end

    # ...
  end

  # good
  describe ArticlesController do
    #...

    describe 'GET new' do
      it 'assigns a new article' do
        get :new
        expect(assigns[:article]).to be_a(Article)
      end

      it 'renders the new article template' do
        get :new
        expect(response).to render_template :new
      end
    end
  end
  ```

* <a name="context-cases"></a>
  `context` blocks should pretty much always have an opposite negative case. It
  should actually be a strong code smell if there is a single context (without a
  matching negative case) that it needs refactoring, or may have no purpose.
<sup>[[link](#context-cases)]</sup>

  ```ruby
  # bad - needs refactoring
  describe '#attributes' do
    context 'the returned hash' do
      it 'includes the display name' do
        # ...
      end

      it 'includes the creation time' do
        # ...
      end
    end
  end

  # bad - the negative case needs to be tested, but isn't
  describe '#attributes' do
    context 'when display name is present' do
      before do
        subject.display_name = 'something'
      end

      it 'includes the display name' do
        # ...
      end
    end
  end

  # good
  describe '#attributes' do
    subject { FactoryBot.create(:article) }

    specify do
      expect(subject.attributes).to include subject.display_name
      expect(subject.attributes).to include subject.created_at
    end
  end

  describe '#attributes' do
    context 'when display name is present' do
      before do
        subject.display_name = 'something'
      end

      it 'includes the display name' do
        # ...
      end
    end

    context 'when display name is not present' do
      before do
        subject.display_name = nil
      end

      it 'does not include the display name' do
        # ...
      end
    end
  end
  ```

* <a name="let-blocks"></a>
  Use `let` blocks instead of `before` blocks to create data for the spec
  examples. `let` blocks get lazily evaluated. It also removes the instance
  variables from the test suite (which don't look as nice as local variables).
<sup>[[link](#let-blocks)]</sup>

  These should primarily be used when you have duplication among a number of
  `it` blocks within a `context` but not all of them. Be careful with overuse of
  `let` as it makes the test suite much more difficult to read.

  ```ruby
  # bad
  before { @article = FactoryBot.create(:article) }

  # good
  let(:article) { FactoryBot.create(:article) }
  ```

* <a name="use-subject"></a>
  Use `subject` when possible
<sup>[[link](#use-subject)]</sup>

  ```ruby
  describe Article do
    subject { FactoryBot.create(:article) }

    it 'is not published on creation' do
      expect(subject).not_to be_published
    end
  end
  ```

* <a name="it-in-iterators"></a>
  Do not write iterators to generate tests. When another developer adds a
  feature to one of the items in the iteration, he must then break it out into a
  separate test - he is forced to edit code that has nothing to do with his pull
  request.
<sup>[[link](#it-in-iterators)]</sup>

  ```ruby
  # bad
  [:new, :show, :index].each do |action|
    it 'returns 200' do
      get action
      expect(response).to be_ok
    end
  end

  # good - more verbose, but better for the future development
  describe 'GET new' do
    it 'returns 200' do
      get :new
      expect(response).to be_ok
    end
  end

  describe 'GET show' do
    it 'returns 200' do
      get :show
      expect(response).to be_ok
    end
  end

  describe 'GET index' do
    it 'returns 200' do
      get :index
      expect(response).to be_ok
    end
  end
  ```

* <a name="incidental-state"></a>
  Avoid incidental state as much as possible.
<sup>[[link](#incidental-state)]</sup>

  ```ruby
  # bad
  it 'publishes the article' do
    article.publish

    # Creating another shared Article test object above would cause this
    # test to break
    expect(Article.count).to eq(2)
  end

  # good
  it 'publishes the article' do
    expect { article.publish }.to change(Article, :count).by(1)
  end
  ```

* <a name="dry"></a>
  Be careful not to focus on being 'DRY' by moving repeated expectations into a
  shared environment too early, as this can lead to brittle tests that rely too
  much on one other.
<sup>[[link](#dry)]</sup>

  It general it is best to start with doing everything directly in your `it`
  blocks even if it is duplication and then refactor your tests after you have
  them working to be a little more DRY. However, keep in mind that duplication
  in test suites is NOT frowned upon, in fact it is preferred if it provides
  easier understanding and reading of a test.

* <a name="factories"></a>
  Use [Factory Bot](https://github.com/thoughtbot/factory_bot) to create test
  objects in integration tests. You should very rarely have to use
  `ModelName.create` within an integration spec. Do **not** use fixtures as they
  are not nearly as maintainable as factories.
<sup>[[link](#factories)]</sup>

  ```ruby
  subject(:article) { FactoryBot.create(:some_article) }
  ```

* <a name="doubles"></a>
  Use mocks and stubs with caution. While they help to improve the performance
  of the test suite, you can mock/stub yourself into a false-positive state very
  easily. When resorting to mocking and stubbing, only mock against a small,
  stable, obvious (or documented) API, so stubs are likely to represent reality
  after future refactoring.
<sup>[[link](#doubles)]</sup>

  This generally means you should use them with more isolated/behavioral
  tests rather than with integration tests.

  ```ruby
  # double an object
  article = double('article')

  # stubbing a method
  allow(Article).to receive(:find).with(5).and_return(article)
  ```

  *NOTE*: if you stub a method that could give a false-positive test result, you
  have gone too far. See below:

  ```ruby
  # bad
  subject { double('article') }

  describe '#summary' do
    context 'when summary is not present' do
      # This stubbing of the #nil? method, makes the test pass, but
      # you are no longer testing the functionality of the code,
      # you are testing the functionality of the test suite.
      # This test would pass if there was not a single line of code
      # written for the Article class.
      it 'returns nil' do
        summary = double('summary')
        allow(subject).to receive(:summary).and_return(summary)
        allow(summary).to receive(:nil?).and_return(true)
        expect(subject.summary).to be_nil
      end
    end
  end

  # good
  subject { double('article') }

  describe '#summary' do
    context 'when summary is not present' do
      # This is no longer stubbing all of the functionality, and will
      # actually test the objects handling of the methods return value.
      it 'returns nil' do
        allow(subject).to receive(:summary).and_return(nil)
        expect(subject.summary).to be_nil
      end
    end
  end
  ```

* <a name="dealing-with-time"></a>
  Always use [Timecop](https://github.com/travisjeffery/timecop) instead of
  stubbing anything on Time or Date.
<sup>[[link](#dealing-with-time)]</sup>

  ```ruby
  # bad
  it 'offsets the time 2 days into the future' do
    current_time = Time.now
    allow(Time).to receive(:now).and_return(current_time)
    expect(subject.get_offset_time).to eq(current_time + 2.days)
  end

  # good
  it 'offsets the time 2 days into the future' do
    Timecop.freeze(Time.now) do
      expect(subject.get_offset_time).to eq 2.days.from_now
    end
  end
  ```

## Naming

* <a name="context-descriptions"></a>
  `context` block descriptions should always start with 'when', 'with' or
  'without', and be in the form of a sentence (or form a part of a sentence)
  with proper grammar.
<sup>[[link](#context-descriptions)]</sup>

  ```ruby
  # bad
  context 'the display name not present' do
    # ...
  end

  context 'the display name has utf8 characters' do
    # ...
  end

  context 'the display name has no utf8 characters' do
    # ...
  end

  # good
  context 'when the display name is not present' do
    # ...
  end

  context 'with utf8 characters in the display name' do
    # ...
  end

  context 'without utf8 characters in the display name' do
    # ...
  end
  ```

* <a name="example-descriptions"></a>
  `it`/`specify` block descriptions should never end with a conditional. This
  is a code smell that the `it` most likely needs to be wrapped in a `context`.
<sup>[[link](#example-descriptions)]</sup>

  ```ruby
  # bad
  it 'returns the display name if it is present' do
    # ...
  end

  # good
  context 'when display name is present' do
    it 'returns the display name' do
      # ...
    end
  end

  # This encourages the addition of negative test cases that might have
  # been overlooked
  context 'when display name is not present' do
    it 'returns nil' do
      # ...
    end
  end
  ```

* <a name="example-group-naming"></a>
  Prefix `describe` description with a hash for instance methods, with a
  dot for class methods.
<sup>[[link](#example-group-naming)]</sup>

  Given the following exists

  ```ruby
  class Article
    def summary
      # ...
    end

    def self.latest
      # ...
    end
  end
  ```

  ```
  # bad
  describe Article do
    describe 'summary' do
      #...
    end

    describe 'latest' do
      #...
    end
  end

  # good
  describe Article do
    describe '#summary' do
      #...
    end

    describe '.latest' do
      #...
    end
  end
  ```

## Matchers

* <a name="predicate-matchers"></a>
  Use RSpec's predicate matcher methods when possible.
<sup>[[link](#predicate-matchers)]</sup>

  ```ruby
  # bad
  it 'is published' do
    expect(subject.published?).to be true
  end

  # good
  it 'is published' do
    expect(subject).to be_published
  end
  ```

* <a name="be-matcher"></a>
  Avoid using `be` matcher without arguments. It is too generic, as it pass on
  everything that is not `nil` or `false`. If that is the exact intend, use
  `be_truthy`. In all other cases it's better to specify what exactly is the
  expected value.
<sup>[[link](#be-matcher)]</sup>

  ```ruby
  # bad
  it 'has author' do
    expect(article.author).to be
  end

  # good
  it 'has author' do
    expect(article.author).to be_truthy # same as the original
    expect(article.author).not_to be_nil # `be` is often used to check for non-nil value
    expect(article.author).to be_an(Author) # explicit check for the type of the value
  end
  ```

## Rails

### Views

* <a name="view-directory-structure"></a>
  The directory structure of the view specs `spec/views` matches the
  one in `app/views`. For example the specs for the views in
  `app/views/users` are placed in `spec/views/users`.
<sup>[[link](#view-directory-structure)]</sup>

* <a name="view-spec-file-name"></a>
  The naming convention for the view specs is adding `_spec.rb` to the
  view name, for example the view `_form.html.erb` has a
  corresponding spec `_form.html.erb_spec.rb`.
<sup>[[link](#view-spec-file-name)]</sup>

* <a name="view-require-spec-helper"></a>
  `spec_helper.rb` needs to be required in each view spec file.
<sup>[[link](#view-require-spec-helper)]</sup>

* <a name="view-outer-describe"></a>
  The outer `describe` block uses the path to the view without the
  `app/views` part. This is used by the `render` method when it is
  called without arguments.
<sup>[[link](#view-outer-describe)]</sup>

  ```ruby
  # spec/views/articles/new.html.erb_spec.rb
  require 'spec_helper'

  describe 'articles/new.html.erb' do
    # ...
  end
  ```

* <a name="view-mock-models"></a>
  Always mock the models in the view specs. The purpose of the view is
  only to display information.
<sup>[[link](#view-mock-models)]</sup>

* <a name="view-assign"></a>
  The method `assign` supplies the instance variables which the view
  uses and are supplied by the controller.
<sup>[[link](#view-assign)]</sup>

  ```ruby
  # spec/views/articles/edit.html.erb_spec.rb
  describe 'articles/edit.html.erb' do
    it 'renders the form for a new article creation' do
      assign(:article, double(Article).as_null_object)
      render
      expect(rendered).to have_selector('form',
        method: 'post',
        action: articles_path
      ) do |form|
        expect(form).to have_selector('input', type: 'submit')
      end
    end
  end
  ```

* <a name="view-capybara-negative-selectors"></a>
  Prefer the capybara negative selectors over `to_not` with the positive.
<sup>[[link](#view-capybara-negative-selectors)]</sup>

  ```ruby
  # bad
  expect(page).to_not have_selector('input', type: 'submit')
  expect(page).to_not have_xpath('tr')

  # good
  expect(page).to have_no_selector('input', type: 'submit')
  expect(page).to have_no_xpath('tr')
  ```

* <a name="view-helper-stub"></a>
  When a view uses helper methods, these methods need to be stubbed. Stubbing
  the helper methods is done on the `template` object:
<sup>[[link](#view-helper-stub)]</sup>

  ```ruby
  # app/helpers/articles_helper.rb
  class ArticlesHelper
    def formatted_date(date)
      # ...
    end
  end
  ```

  ```Rails
  # app/views/articles/show.html.erb
  <%= 'Published at: #{formatted_date(@article.published_at)}' %>
  ```

  ```ruby
  # spec/views/articles/show.html.erb_spec.rb
  describe 'articles/show.html.erb' do
    it 'displays the formatted date of article publishing' do
      article = double(Article, published_at: Date.new(2012, 01, 01))
      assign(:article, article)

      allow(template).to_receive(:formatted_date).with(article.published_at).and_return('01.01.2012')

      render
      expect(rendered).to have_content('Published at: 01.01.2012')
    end
  end
  ```

* <a name="view-helpers"></a>
  The helpers specs are separated from the view specs in the `spec/helpers`
  directory.
<sup>[[link](#view-helpers)]</sup>

### Controllers

* <a name="controller-models"></a>
  Mock the models and stub their methods. Testing the controller should not
  depend on the model creation.
<sup>[[link](#controller-models)]</sup>

* <a name="controller-behaviour"></a>
  Test only the behaviour the controller should be responsible about:
<sup>[[link](#controller-behaviour)]</sup>

  * Execution of particular methods
  * Data returned from the action - assigns, etc.
  * Result from the action - template render, redirect, etc.

  ```ruby
  # Example of a commonly used controller spec
  # spec/controllers/articles_controller_spec.rb
  # We are interested only in the actions the controller should perform
  # So we are mocking the model creation and stubbing its methods
  # And we concentrate only on the things the controller should do

  describe ArticlesController do
    # The model will be used in the specs for all methods of the controller
    let(:article) { double(Article) }

    describe 'POST create' do
      before { allow(Article).to receive(:new).and_return(article) }

      it 'creates a new article with the given attributes' do
        expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
        post :create, message: { title: 'The New Article Title' }
      end

      it 'saves the article' do
        expect(article).to receive(:save)
        post :create
      end

      it 'redirects to the Articles index' do
        allow(article).to receive(:save)
        post :create
        expect(response).to redirect_to(action: 'index')
      end
    end
  end
  ```

* <a name="controller-contexts"></a>
  Use context when the controller action has different behaviour depending on
  the received params.
<sup>[[link](#controller-contexts)]</sup>

  ```ruby
  # A classic example for use of contexts in a controller spec is creation or update when the object saves successfully or not.

  describe ArticlesController do
    let(:article) { double(Article) }

    describe 'POST create' do
      before { allow(Article).to receive(:new).and_return(article) }

      it 'creates a new article with the given attributes' do
        expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
        post :create, article: { title: 'The New Article Title' }
      end

      it 'saves the article' do
        expect(article).to receive(:save)
        post :create
      end

      context 'when the article saves successfully' do
        before do
          allow(article).to receive(:save).and_return(true)
        end

        it 'sets a flash[:notice] message' do
          post :create
          expect(flash[:notice]).to eq('The article was saved successfully.')
        end

        it 'redirects to the Articles index' do
          post :create
          expect(response).to redirect_to(action: 'index')
        end
      end

      context 'when the article fails to save' do
        before do
          allow(article).to receive(:save).and_return(false)
        end

        it 'assigns @article' do
          post :create
          expect(assigns[:article]).to eq(article)
        end

        it 're-renders the 'new' template' do
          post :create
          expect(response).to render_template('new')
        end
      end
    end
  end
  ```

### Models

* <a name="model-mocks"></a>
  Do not mock the models in their own specs.
<sup>[[link](#model-mocks)]</sup>

* <a name="model-objects"></a>
  Use `FactoryBot.create` to make real objects, or just use a new (unsaved)
  instance with `subject`.
<sup>[[link](#model-objects)]</sup>

  ```ruby
  describe Article do
    let(:article) { FactoryBot.create(:article) }

    # Currently, 'subject' is the same as 'Article.new'
    it 'is an instance of Article' do
      expect(subject).to be_an Article
    end

    it 'is not persisted' do
      expect(subject).to_not be_persisted
    end
  end
  ```

* <a name="model-mock-associations"></a>
  It is acceptable to mock other models or child objects.
<sup>[[link](#model-mock-associations)]</sup>

* <a name="model-avoid-duplication"></a>
  Create the model for all examples in the spec to avoid duplication.
<sup>[[link](#model-avoid-duplication)]</sup>

  ```ruby
  describe Article do
    let(:article) { FactoryBot.create(:article) }
  end
  ```

* <a name="model-check-validity"></a>
  Add an example ensuring that the FactoryBot.created model is valid.
<sup>[[link](#model-check-validity)]</sup>

  ```ruby
  describe Article do
    it 'is valid with valid attributes' do
      expect(article).to be_valid
    end
  end
  ```

* <a name="model-validations"></a>
  When testing validations, use `expect(model.errors[:attribute].size).to eq(x)` to specify the attribute
  which should be validated. Using `be_valid` does not guarantee that the
  problem is in the intended attribute.
<sup>[[link](#model-validations)]</sup>

  ```ruby
  # bad
  describe '#title' do
    it 'is required' do
      article.title = nil
      expect(article).to_not be_valid
    end
  end

  # preferred
  describe '#title' do
    it 'is required' do
      article.title = nil
      article.valid?
      expect(article.errors[:title].size).to eq(1)
    end
  end
  ```

* <a name="model-separate-describe-for-attribute-validations"></a>
  Add a separate `describe` for each attribute which has validations.
<sup>[[link](#model-separate-describe-for-attribute-validations)]</sup>

  ```ruby
  describe '#title' do
    it 'is required' do
      article.title = nil
      article.valid?
      expect(article.errors[:title].size).to eq(1)
    end
  end

  describe '#name' do
    it 'is required' do
      article.name = nil
      article.valid?
      expect(article.errors[:name].size).to eq(1)
    end
  end
  ```

* <a name="model-name-another-object"></a>
  When testing uniqueness of a model attribute, name the other object
  `another_object`.
<sup>[[link](#model-name-another-object)]</sup>

  ```ruby
  describe Article do
    describe '#title' do
      it 'is unique' do
        another_article = FactoryBot.create(:article, title: article.title)
        article.valid?
        expect(article.errors[:title].size).to eq(1)
      end
    end
  end
  ```

### Mailers

* <a name="mailer-mock-model"></a>
  The model in the mailer spec should be mocked. The mailer should not depend on
  the model creation.
<sup>[[link](#mailer-mock-model)]</sup>

* <a name="mailer-expectations"></a>
  The mailer spec should verify that:
<sup>[[link](#mailer-expectations)]</sup>

  * the subject is correct
  * the receiver e-mail is correct
  * the e-mail is sent to the right e-mail address
  * the e-mail contains the required information

  ```ruby
  describe SubscriberMailer do
    let(:subscriber) { double(Subscription, email: 'johndoe@test.com', name: 'John Doe') }

    describe 'successful registration email' do
      subject { SubscriptionMailer.successful_registration_email(subscriber) }

      its(:subject) { should == 'Successful Registration!' }
      its(:from) { should == ['info@your_site.com'] }
      its(:to) { should == [subscriber.email] }

      it 'contains the subscriber name' do
        expect(subject.body.encoded).to match(subscriber.name)
      end
    end
  end
  ```

# Contributing

Open tickets or send pull requests with improvements. Please write [good commit messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
or your pull requests will be closed.

# Credit

Inspiration was taken from the following:

[HowAboutWe's RSpec style guide](https://github.com/howaboutwe/rspec-style-guide)

[Community Rails style guide](https://github.com/rubocop-hq/rails-style-guide)
