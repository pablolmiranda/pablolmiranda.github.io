Building your tests as a product
================================

The usual scenario is to put all your energy over your production code, because as the name says it is the code the will be used to thousand of people. The collateral effect of this is that you spend the minimal effort over your tests, creating a test code that sometimes doesn't reflex the intension of the code or even the system.

To prove my point let's say that we have a contact form with name, email, subject and text fields.

Using [Watir](http://watir.com/) test library you can create a simple test to send the content of the contact form as below:

```
    browser = Watir::Browser.new :firefox
    browser.text_field(:name => 'contact.name').set = 'Test User'
    browser.text_field(:name => 'contact.email').set = 'test@test.com'
    browser.select_list(:name => 'contact.subject').select 'Improvements'
    browser.text_field(:name => 'contact.text').set 'We need to improve this example'
    browser.button(:name => 'contact.send_button').click
```

This is a good and simple example on how we can test a contact form, but it is not so maintainable because we have direct access to the browser elements and operations. We can improve a little bit if we start to use the [Page Object](https://github.com/cheezy/page-object) library. The Page Object abstracts the operations over the structure of the page. Let's give a try:

```
    class ContactFormPageObject
        text_field(:name, :id => 'contact.name')
        text_field(:email, :id => 'contact.email')
        select_list(:subject, :id => 'contact.subject')
        text_area(:text, :id => 'contact.text')
        button(:send_button, :id => 'contaxt.send_button')
    end

    browser = Watir::Browser.new :firefox
    contact_form = ContactFormPageObject.new browser
    contact_form.name = 'Test User'
    contact_form.email = 'test@test.com'
    contact_form.subject = 'Improvements'
    contact_form.text = 'We need to improve this example'
    contact_form.send_button
```

The example if better now, we only know that we are interacting over a form, and if you remove the browser reference, can be over any kind of UI. That's little that we can improve here because this example is very simple, maybe the only think is encapsulate the send button over a object command method like **send!**

But what happens with you have a more complex flow like text the Yahoo! Homepage news carousel:

```
    browser = Watir::Browser.new :firefox
    homepage = Homepage.new browser
    
    homepage.carousel.next!
    expect(homepage.carousel).to be_updated # maps to homepage.carousel.updated?
    expect(homepage.carousel.highlighted_news).to be == homapage.carousel.first

    homepage.carousel.hover!(:first)
    expect(homepage.carousel.highlighted_news).to be == homapage.carousel.news(:first)

    homepage.carousel.hover!(:second)
    expect(homepage.carousel.highlighted_news).to be == homapage.carousel.news(:second)

    homepage.carousel.hover!(:last)
    expect(homepage.carousel.highlighted_news).to be == homapage.carousel.news(:last)

    homepage.carousel.previous!
    expect(homepage.carousel).to be_updated # maps to homepage.carousel.updated?
    expect(homepage.carousel.highlighted_news).to be == homapage.carousel.first
```

This example create a clean interface to our visual component avoiding any kind of knowledge about the internal structure of the component. All the access are through command methods and query methods, and how this methods interaction with the UI elements are not important anymore and if you want to reuse the visual component you can reuse the test too. Now your test code follow the same rules of your production code.

You should think about your tests just like you think about the code of other parts of the system, it should be maintainable, well structured, reflex the system abstraction in the correct level and more important: reflex the correct behavior of the system. Your tests are the live specification of your system, so work to make them healthy.