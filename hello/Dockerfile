FROM ruby

WORKDIR /usr/src

ADD . .

WORKDIR /usr/src/hello
RUN gem install rails
RUN bundle install

CMD ["/usr/local/bundle/bin/rails", "s"]
