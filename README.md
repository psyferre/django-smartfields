
django-smart-fields
===================

Django Model Fields that are smart. Currently only file fields.

Features
--------

* Cleanup of orphan files after the entry in database was deleted or if file was replaced with a different one
* Image files conversion and/or resizing (requires PIL)
* Video files conversion (requires avconv or ffmpeg)
* Automatic addition of the converted FileFields to the model. (For instance if a particular ImageField defined in the model with a name 'image', using custom settings for that field named 'png' each instance of that model will be initialized with an addidional field 'image_png'.)
* Custom file upload form fields that feature:
    * applying default images from static files if image was not uploaded yet.
    * displaying of initial value (displaying currently saved image or video)
    * upload progress (requires plupload, works in all browsers)
    * file size display and limit before uploading (requires plupload)
* Custom mini file management (requires plupload)
    * queueing of multiple files to be uploaded
    * upload progress and file size for each file and total
    * representing uploaded files as list with a delete button

Dependancies
------------
* `Django <https://docs.djangoproject.com/en/dev/>`_ (currently works with development 1.6 version and possibly with 1.5 RC, couple methods used for widget rendering needs to be ported from development verision for it to work in earlier version <= 1.4) 
    * If anyone is willing to make it work with django 1.4 or less, will be much appreciated, since it would be a hassle for me due to the project I am working on. Send me a message if assistance needed with that task.
* `Python PIL <http://pypi.python.org/pypi/PIL>`_ used for image conversion/resizing
* `avconv <http://libav.org/avconv.html>`_ for video conversion (should also work with ffmpeg, will require settings modifications though. has not been tested with ffmpeg yet)
* `Plupload <http://www.plupload.com/>`_ for using smartfields in forms (free for development, license for production is ~$14. Totally worth it.)

How to use it
-------------

The best way to demonstrate features is through examples.
For any of the features to work you have to extend the base abstract model ``smart_fields.models.SmartFieldsBaseModel``. If for some reason it is not feasible (lets say you are using GeoDjango or something) you can extend ``smart_fields.models.SmartFieldsHandler`` and add required methods. Here is an example:

    class SmartFieldsModel(models.Model, smart_fields.models.SmartFieldsHandler):
        ... 
	fields
	...

        def __init__(self, *args, **kwargs):
            super(SmartFieldsModel, self).__init__(*args, **kwargs)
            self.smart_fields_init() ## !important. after the super call

        def save(self, old=None, *args, **kwargs):
            super(SmartFieldsModel, self).save(*args, **kwargs)
            self.smart_fields_save()  ## !important. after the super call

        def delete(self, *args, **kwargs):
            self.smart_fields_delete()  ## !important. before the super call
            super(SmartFieldsModel, self).delete(*args, **kwargs)

Now we need to specify what settings needs to be applied and to which fields. Here is an example:


    class SmartFieldsModel(smart_fields.models.SmartFieldsBaseModel):
        def image_upload_url(instance):
            return reverse("uploads:images", kwargs={'pk': instance.pk})

        user = models.ForeignKey(User, editable=False)
        image = models.CrowdSmartImageField(upload_to="place/for/images/", upload_url=image_upload_url, keep_orphans=True)
        ...

        @property
        def smart_fields_settings(self): # required
            return {
                'image': { # <- name of the field. required
                    'instance': self.image, # <- instance of the field. required
                    'profile': { # settings for converting/resizing
                        'big': {
                            'dimensions': (320, 200),
                            'format': 'PNG',
                            'default': "default/images/320_200.png"
                            },
                        'small': {
                            'dimensions': (160, 100),
                            'format': 'JPG',
                            'default': "default/images/160_100.jpg"
                            },
                        },
                    },

In the example above each instance of SmartFieldsModel will have:
* instance.image with an original file uploaded, or default image if no image has been uploaded
* instance.image_big and instance.image_small which are instances of FileField, despite that there are no actual columns for them in the database, we can still use them as regular model fields. For instance we can use them in templates: ``{{ instance.image_big.url }}``. And as you would expect it will give you a url to a copy of an original image in png format with dimensions 320px by 200px.

From now on I will refer to the above required property as <strong>smart_field_settings</strong>

Above functionality is not limited to only one file field per model, more over it is possible to mix it with any fields that are derived from FileField (SmartImageField, SmartVideoField, ... more to come). Although, if any feature but orphan file cleanup is desired on the field, the have to be SmartFileField, SmartImageField or SmartVideoField types.

Rest of the featues usage and settings explanations hopefully is coming soon with documentation.