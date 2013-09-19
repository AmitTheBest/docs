# DAY 8: THE FORMS

Perhaps your first impression of Agile Toolkit when you saw a form on the main page of [agiletoolkit.org](agiletoolkit.org "agiletoolkit.org"). Forms is a very important component in Agile Toolkit. Form is in the center of interactivity and Agile Toolkit is all about interactivity. You might have made forms before and you might know how tedious task it is. Some tools might have helped you to solve part of the problem with dealing with form, such as displaying or validating them. Agile Toolkit goes all the way when it comes to form starting from defining form fields and ending with reading field values.

Forms in Agile Toolkit does not require you to know anything about the old way of making forms. Once you start using Forms in Agile Toolkit, you wouldn't want to go back to the old ways. To remind of how simply it is, let me place a code here from Agile Toolkit main page. And I'd like to stress: that's ALL the code you need to write to make a form. There are no hidden CSS or JavaScript which you need to fiddle with.

### Result

    HERE SHOULD BE AN EXAMPLE
 	
### Code
    $f=$p->add('Form');
    $f->addField('line','name')->validateNotNull();
    $f->addField('line','surname');
    $f->addSubmit();
    if($f->isSubmitted()){
        $f->js()->univ()
            ->alert('Thank you, '.$f->get('name').
            ' '.$f->get('surname'))
            ->execute();
    }

## Adding a form
----
Like anything else, form in Agile Toolkit is a class. Inside the form you can add all sorts of other objects, fields, buttons, hint, links, grids or anything else. Agile Toolkit does not separate the visual and functional part of a form, but instead everything is seamlessly integrated.

To use form, you do not need to create a new form class. Like in example above, you can just use standard form, add fields into it and it's good enough. However if you find yourself using the same set of initialization lines for a similar type of form, you might need to bundle it into a standalone class.

### Form + Model
There is a way to build a form based on your model data. Because we already made a couple of models, it's much more time-efficient to use this approach.

    $p->add('Form')->setModel('Job');

When you called setModel on a newly created form, it automatically populated all editable fields from model Job. Let's quickly create a page for adding a new Job. Instead of adding a new page class, let's add a sub-page to page_jobs class:

    function init(){
        parent::init();
        $this->api->stickyGET('token');
    }
    function jobEditPage($token=null){
        $m=$f->setModel('Job');

        // Load data if token is specified
        if($token){
            $row=$m->getBy('token',$token);
            if(!$row)throw $this->exception('Invalid token');
            $m->tryLoad($row['id']);
        }

        $f=$this->add('Form');
        $f->setModel($m);
        $f->onSubmit(function($f){
            $f->update();
            return $f->js()->univ()->successMessage('Job Added');
        });
    }
    function page_edit(){
        return $this->jobEditPage($_GET['token']);
    }
    function page_new(){
        $this->api->stickyForget('token');
        return $this->jobEditPage();
    }
We actually did few things here. First we declared init() method for the page (which is executed for main and all the subpages) and told API that $_GET['token'] is a significant variable and it should be carried along when building links. This way we do not need to worry about passing this argument between preview and editing forms.

Other thing we placed code for form generation into a jobEditPage() function. This code is called from both "new" and "edit" page. "new" page however specifically drops token

Then open http://localhost/jobeet/jobs/new.html in your browser. You will notice how fields which are declared as system() in the model are not shown on the form. Did you notice that we used PHP 5.3+ syntax for form submission? It has an added benefit of being able to place multiple onSubmit handlers on the form and it also automatically catches some exceptions and displays them.

## Validation
----
Before adding validation, you need to decide if this should be a Model-level validation or Form level. If you go with Model-level than validation will be always performed even if you don't use UI or created a RESTful API. Form-level validation will only be used on this page. To demonstrate both approaches, let's set a minimum length for field company through Form validation, however we will check format of the URL through model-level check.

### Form-level Validation
If you have looked at [Form Documentation](http://agiletoolkit.org/form/validation "Form Documentation"), you know that there are many ways how to perform a validation. We will use validation inside our submit handler by adding the following line before "$f->update()":

        if(strlen($f->get('company'))<5){
            throw $f->exception('Company name is too short')->setField('company');
        }
Next, open Model/Job.php and let's review the fields and add some validation on Model level. Here is the field definitions after review:

    function init(){
        parent::init();

        $this->addField('category_id')
            ->hasOne('Model_Category')
            ->caption('Category')
            ;

        $this->addField('type')
            ;

        $this->addField('company')
            ->mandatory(true)
            ;

        $this->addField('logo')
            ;

        $this->addField('url')
            ->caption('URL')
            ->mandatory(true)
            ->validate(function($val){
                if(!filter_var($val,FILTER_VALIDATE_URL,FILTER_FLAG_SCHEME_REQUIRED)){
                    return 'Wrong URL';
                }
            })
            ;

        $this->addField('position')
            ->mandatory(true)
            ;

        $this->addField('location')
            ->mandatory(true)
            ;

        $this->addField('description')
            ->datatype('text')
            ->mandatory(true)
            ;

        $this->addField('how_to_apply')
            ->datatype('text')
            ->mandatory(true)
            ;

        $this->addField('is_public')
            ->datatype('boolean')
            ->caption('Public?')
            ;

        $this->addField('is_activated')
            ->datatype('boolean')
            ->system(true)
            ;

        $this->addField('email')
            ->mandatory(true)
            ;

        // System fields
        $this->addField('token')
            ->system(true)
            ;

        
        $this->addField('created_dts')
            ->datatype('datetime')
            ->system(true)
            ;

        $this->addField('updated_dts')
            ->datatype('datetime')
            ->system(true)
            ;

        $this->addField('expires_at')
            ->datatype('date')
            ->system(true)
            ;
    }
You will notice that I have also made definitions into multi-line statements. This improves readability and helps to manipulate file easier. Next let's change type into radio buttons with several possible values. (Use datatype=list for drop-down)

Agile Toolkit is usually pretty good at generating nice captions from the field names. For example how_to_apply is converted into "How To Apply" label. In some cases I have added ->caption() which changes field label. I have also added system(true) for expiration date and is_activated fields.

When you see the form in your browser next time, another thing you would probably notice is that mandatory fields are marked with red stars. That is the default behaviour although like many other things it can be changed too.

        $this->addField('type')
            ->datatype('radio')
            ->listData(array(
                        'full-time'=>'Full Time',
                        'part-time'=>'Part Time',
                        'freelance'=>'Freelance',
                        ))
            ->defaultValue('full-time')
            ;

Note that this field type will automatically make sure that one of the values is used, therefore we do not need to worry about validating this field any further.

## Uploads and Filestore
----
File uploads is ofter an awful experience in web developer's daily life. That is because apart from file itself, it's associated with quite a few bits of information. Then there is a problem of how to store files. If project hosts a lot of uploads, how to avoid directories from growing insanely large? In most cases that's a task of developer to take care of that. What about original filename and mime types? Agile Toolkit does this for you.

Agile Toolkit introduces an add-on called Filestore. It integrates with file uploads and stores them into a central directory under a hashed structure and using random file names. At the same time it adds a record into a filestore_file table through a Model_Filestore_File, where it records type, original filename, size, randomly generated filename where data is stored.

To use filestore, you will need 3 things. First you will need to create tables in your database. Locate file atk4-addons/misc/docs/filestore.001.sql and import into your database. This will create few tables for you. Second thing you will need is directory to host uplodas. Create "upload" directory in your webroot and make it writable. Third thing you would need is some sort of management console for your datastore. Create file page/filestore.php with the following content:

    class page_filestore extends Page_Filestore_FileAdmin {}
This page will come with a sample upload element, which you can try with one of the allowed types - gif, png or jpeg. Once you verify this all, you can change definition for logo field:

        $this->addField('logo')
            ->HasOne('Model_Filestore_File')
            ->displaytype('file')
            ;
Because forms in Agile Toolkit are submitted through AJAX, file upload is started immediately after file is selected. Even if you do not submit the form, file will be there in the filestore.

## Generating Token automatically
----
When new job is added - token should be generated which then is used for editing. We have already defined beforeInsert method in the model, so we just need to add the following code at the end of this method:

        $data['token']=sha1($data['email'].rand(11111, 99999));

## Showing Preview and Editing again
----
When form editing is complete, we need to show user a preview of the job, how it would appear to the user. Lets return to our page/jobs.php and do some changes there. First after form is successfully submitted let's redirect user to a preview page. Replace return statement inside onSubmit handler's function with this:

            return $f->js()->univ()->location($f->api->url('../preview',
                    array('token'=>$f->get('token'))));
Next, add a preview sub-page:

    function page_preview(){
        $m=$this->add('Model_Job_Public');

        // Load token data
        $row=$m->getBy('token',$_GET['token']);
        if(!$row)throw $this->exception('Invalid token');
        $m->tryLoad($row['id']);

        $v=$this->add('View',null,null,array('view/job_details'));
        $v->setModel($m);

        $v->add('Button',null,'Buttons')->setLabel('Edit')->js('click')->univ()->location(
            $this->api->url('../edit'));

        if($v->add('Button',null,'Buttons')->setLabel('Publish')->isClicked('Are you sure?')){
            $m->set('is_public',true)->save();
            $parts=array(
                    $m->get('location'),
                    $m->get('company'),
                    $m->get('id'),
                    $m->get('position'),
                    );

            $parts=preg_replace('/[^a-zA-Z 0-9-]/','',$parts);
            $parts=preg_replace('/^$/','-',$parts);
            $parts=str_replace(' ','_',$parts);
            $page=implode('/',$parts);

            $this->js()->univ()->location($this->api->url(
                        'job/'.$page,array('token'=>false)))->execute();
        }
        
    }
Yet another amazing use of integrated AJAX is button's isClicked() function. The code is so simple I don't even need to explain what it does. Be sure to always call execute() on js() chain when using with isClicked().

Today we did some web-magic! You learned about all sorts of validations, form manipulating and other things. Because forms are so simple in Agile Toolkit you probably have a lot of time today which you should spend on doing some refactoring. I will give you two hints on what to refactor:

* link generation to job view pages appears on page_jobs.php and lib/CategoryJobs.php files. Move this function into Model/Job.php
* loading model by token was done in page_preview and jobEditPage() functions. Move it into Model/Job and call loadByToken(). Remember, getXXXX() returns array, loadXXXX loads data but returns same object ($this).

Tomorrow we will start working on Administration System for our job portal.