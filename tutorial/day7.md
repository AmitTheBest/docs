# DAY 7: CATEGORY PAGE

Yesterday you expanded your knowledge of Agile Toolkit in a lot of different areas: querying with Models, Population scripts, debugging, and custom configuration. But most importantly you should see how important to separate business logic from the front-end. We already have 4 models for our purposes with solid conditions and even some calculated fields which we can virtually apply to ANY view and place them on any page in a matter of few lines.

I hope you worked on the Jobeet category page as today's tutorial will then be much more valuable for you.

Ready? Let's talk about a possible implementation.

## The Category Page
----
First, we need to add a page to define a pretty URL for the category page. Add this to page/category.php file:

    class page_category extends Page {
        function init(){
            parent::init();
            $p=$this;

            $category=$this->add('Model_Category_Active')->getBy('name',$_GET['name']);

            $p->add('H3')->set($category['name']." (".$category['job_count'].')');
            $jobs = $p->add('JobList');
            $jobs->setModel('Job_Public',array('location','position','company'))
                ->addCondition('category_id',$category['id']);
            $jobs->addFormatter('company','link');
        }
    }

If you try the code at this point, you will find out that JobList class is nowhere to be found. Another point is that we start to repeat some of the UI code and the DRY (don't repeat yourself) principle taught us it's bad. We might have moved JobList outside, however let's create a new View which lists entire category's jobs instead. The code which we are willing to have on our category page would then be:

    class page_category extends Page {
        function init(){
            parent::init();
            $p=$this;

            $category=$this->add('Model_Category_Active')->getBy('name',$_GET['name']);

            $this->add('CategoryJobs')->setCategoryData($category)
                ->template->tryDel('more_link');
        }
    }

The process of moving code between files to improve the structure of the code is called refactoring. Agile Toolkit is a great help when you need to start refactoring. That is because the syntax of the code virtually do not change at all when you move it.

    class CategoryJobs extends View {
        public $jobs=null;
        function setCategoryData($category){
            $p=$this;

            $this->template->set('Category',$category['name'].
                " (".$category['job_count'].')');

            $this->jobs=$p->add('JobList');
            $this->jobs->setModel('Job_Public',array('location','position','company'))
                ->addCondition('category_id',$category['id']);
            $this->jobs->addFormatter('company','link');
            return $this;
        }
    }

    // Move JobList class here

Part of our jobs.php page can also be cleaned up using this new View:


        // skip

        foreach($categories as $category){
            $cj=$this->add('CategoryJobs')->setCategoryData($category);
            $cj->jobs->dq->limit(10);
            if($category['job_count']>10){
                $category['job_count']-=10;
                $cj->template->set($category);
                $cj->template->set('url',
                        $this->api->url('category/'.
                            $category['name']));
            }else{
                $cj->template->del('more_link');
            }
        }   

        // skip

Next, let's give a custom template chunk to your CategoryJobs View by adding function defaultTemplate() into CategoryJobs.php:

    function defaultTemplate(){
        return array('view/category_jobs');
    }
The template could look like this:

    <div id="<?$_name?>" class="category_jobs">
      <h3><?$Category?></h3>
        <?$Content?>  

        <?more_link?>
        <div style="float:right" class="more_link">and <a href="<?$url?>"><?$job_count?></a> more</div>
        <?/?> 
    </div>

Did you notice that we replaced our 'H3' object with setting template's value for $Category directly in CategoryJobs. This would give our designer more flexibility. Also we would need to add a rule to our .htaccess to handle pretty-url requests for category:

    RewriteRule category/([^.]*)?(.html)?$              index.php?page=category&name=$1    [L]

## List Pagination
----
From day 2 requirements: *"The list is paginated with 20 jobs per page."*

To paginate a Grid or List you can just add Paginator class to it. It does two useful things. First it creates a visual pagination controls. Second it manipulates "dsql" object of the view it's attached to to automatically display proper sub-set of data. We need Paginator only on category page, so adding this to page/category.php

        $cj=$this->add('CategoryJobs')->setCategoryData($category);
        $cj->jobs->add('Paginator')->ipp(20);
        $cj->template->tryDel('more_link');
We try to use chaining where possible to minimize amount of variables we are using. When we need more actions on an object, we introduce a short-named variable instead of creating chains like $form->addField()->owner->addField()->owner() and so on. This is to improve code readability. Inside our View however, we used property $jobs instead of the variable. It's a common practice to define public properties. Thanks to that, we can now access $jobs from the page and add a paginator on a grid.

Philosophy of Agile Toolkit says that frontend should be transparent for the frontend. Backend (business logic) should be semi-transparent for the backend. Backend must not assume existance of a frontend (such as session variables, $_GET or $api->page). Never use short variables in backend. If those rules are followed, then code will be clean, very short and very easy to read.

## Sticky Arguments
----
If you try to use paginator now, you will see that clicking on it raises exception. That is because argument "id" is lost. We need to make it persistent, or, sticky. Add the following line before adding of CategoryJobs:

        $this->api->stickyGET('name');
        $cj=$this->add('CategoryJobs')->setCategoryData($category);
        $cj->jobs->add('Paginator')->ipp(20);
        $cj->template->tryDel('more_link');
This however broke the link for the job details, so we need to amend the code by using:

                    $this->api->url('job/'.$page,array('name'=>null)).

StickyGET is not very smart about applying on the objects. Once you call stickyGET, all the url() from there on will pass specified GET argument along.

## See you Tomorrow
----
If you worked on your own implementation yesterday and feel that you didn't learn much today, it means that you are getting used to the way of Agile Toolkit. The process to add a new Page to Agile Toolkit is always the same: Add Page first, put views on the page.... well, that's it. Add some templates if you like. And, if you can apply some good development practices to the mix. If several pages function similarly, inherit them from the same parent. If pages contain similar block of code, create it as a separate view. Always try to make views and try to avoid making functions such as buildForm() on your pages, you are loosing Object-Oriented features if you do.

Tomorrow we will talk about something where Agile Toolkit knows no rivals: Forms.