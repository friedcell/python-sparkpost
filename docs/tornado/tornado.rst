Tornado Web Server
==================

The SparkPost python library comes with an integration with Tornado asynchronous transport.

Use in Tornado
--------------

To use SparkPost in with the AsyncHTTPClient, you have to import SparkPost from sparkpost.tornado instead of sparkpost 
and use yield before all operations as they return a Future.

Standalone use
--------------

.. code-block:: python

    from tornado import gen, ioloop
    from sparkpost.tornado import SparkPost
    
    SPARKPOST_API_KEY = 'API_KEY'
    
    sp = SparkPost(SPARKPOST_API_KEY)
    
    @gen.coroutine
    def send():
        response = yield sp.transmissions.send(
            recipients=['someone@somedomain.com'],
            html='<p>Hello world</p>',
            from_email='test@sparkpostbox.com',
            subject='Hello from python-sparkpost',
            track_opens=True,
            track_clicks=True
        )
        raise gen.Return(response)
    
    ioloop.IOLoop().run_sync(send)

Use in Tornado applications
---------------------------

To use in the context of an application it is best to save the link to an instance of SparkPost on the application and 
then use it in the RequestHandler

.. code-block:: python

    from tornado import gen, ioloop, web
    from sparkpost.tornado import SparkPost

    SPARKPOST_API_KEY = 'API_KEY'

    class SendMailHandler(web.RequestHandler):
        @gen.coroutine
        def send(self, email):
            response = yield self.application.sparkpost.transmissions.send(
                recipients=[email],
                html='<p>Hello world</p>',
                from_email='test@sparkpostbox.com',
                subject='Hello from python-sparkpost',
                track_opens=True,
                track_clicks=True
            )
            raise gen.Return(response)
        
        @gen.coroutine
        def post(self):
            email = self.get_argument("to", None)
            if email:
                response = yield self.send(email)
                self.write(response)

    def make_app():
        app = web.Application([
            (r"/", SendMailHandler),
        ])
        app.sparkpost = SparkPost(SPARKPOST_API_KEY)
        return app

    if __name__ == "__main__":
        app = make_app()
        app.listen(8888)
        ioloop.IOLoop.current().start()

Supported version
-----------------
SparkPost supports the latest Tornado from the 3.2 branch as well as Tornado 4.x branch. Previous versions might work, 
but are not tested against.
