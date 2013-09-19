# DAY 5: THE ROUTING

If you've completed day 4, you should now be familiar with the MVC pattern and it should be feeling like a more and more natural way of coding. Spend a bit more time with it and you won't look back.

Today we will review how Agile Toolkit keeps routing simple yet full-featured

## URLs
----
As you see from the example yesterday, Agile Toolkit relies on a function url() to produce links between pages. It's used both when we click on the job and when we go back to the jobs page. Agile Toolkit has a lightweight yet powerful way to determine pages, link across pages in absolute and relative manner as well as handle arguments in a very unique way.

But first, let's talk about URLs and what exactly they are. In a web context, a URL is the unique identifier of a web resource. When you go to a URL, you ask the browser to fetch a resource identified by that URL. So, as the URL is the interface between the website and the user, it must convey some meaningful information about the resource it references.

Traditionally URL consists of the page itself (jobeet/details.html) and then arguments (?id=123). Some web software applications try to hide everything from there, others try to bloat it with hundred arguments. Some will try to convert them into jobeet/details/id/123 form but to be honest - it only adds complication and does not provide real benefit. That's why Agile Toolkit sticks with default approach of the arguments by default. However there is a way to customize URLs.

### Introducing mod_rewrite
Mod_rewrite is a really fast and flexible way to re-write URLs. In fact if you followed through the day1 you are probably already using it. Look into .htaccess file:

    RewriteEngine on
    RewriteRule ^(index.php.*)$      $1          [L]
    RewriteRule ^[^\.]*$            index.php    [L]
    RewriteRule .html$              index.php    [L]
    
What happens here is very simple. All requests land on the index.php file. Agile Toolkit then determines which page was originally requested from the $_SERVER variable (which contains original request) and sets $api->page variable accordingly. You, however, can re-define this by specifying the page=xx argument. Let's add the following line to .htaccess file right before other RewriteRule:

    RewriteRule jobs/details/(\d+)(-[^.]*)?.html$              index.php?page=jobs_details&id=$1    [L]
    
After adding this, you should be able to access: http://localhost/jobeet/jobs/details/1.html. Easy huh? And what's even more importantly, it's really fast and scalable. However if you play with URLs like that, it will not affect what url() returns. You would need to either generate URLs manually or change the URL class. Usually you would want to have pretty-urls only for a couple of the pages anyway. Usually you would also want to include additional information into the header, such as job title (jobeet/jobs/details/1-Company_blahblah_looking_for_PHP_Developer.html).

### Practical Example

    RewriteRule job/[^/]*/[^/]*/(\d+)/([^.]*)?(.html)?$              index.php?page=jobs_details&id=$1    [L]

This would allow us to use URL such as: http://localhost/jobeet/job/london/agile/1/developer, but let's also modify our Grid to include this link:

    function page_index(){
        $p=$this;

        $jobs = $this->add('Grid');
        $jobs->setModel('Job',array('location','position','company'));
        $jobs->addQuickSearch(array('location','position'));
        $jobs->addButton('Post a Job');
        
        $jobs->addFormatter('company','template')
            ->setTemplate('<a href="'.
                    $this->api->url('job/'.$page)
                    .'">');
    }

Unfortunately if fields $location or $company contain something ugly, Agile Toolkit wouldn't do anything about it, so let's try going to a different route. At this point, the functionality of default "Grid" is insufficient to do what we need and Object-Oriented design of Agile Toolkit comes to help. First let's create our own lister for this page in particular called JobList. Because it's not the class we are going to re-use anywhere else but on this page, it's safe to place it underneath the code of page's class. If you do not like this approach, you can create lib/JobList.php with the same content:

Also return previous formatter to the grid and switch from Grid to our new class inside init()

    class page_jobs extends Page {
        function page_index(){
            $p=$this;

            $jobs = $this->add('JobList');  // changed here
            $jobs->setModel('Job',array('location','position','company'));
            $jobs->addQuickSearch(array('location','position'));
            $jobs->addButton('Post a Job');

            $jobs->addFormatter('company','link'); // changed here
        }
        function page_details(){
            $v=$this->add('View',null,null,array('view/job_details'));
            $m=$v->setModel('Job')->tryLoad($_GET['id']);
            $v->template->del('has_logo');
            $v->add('Button',null,'Buttons')->setLabel('Back')->js('click')->univ()->location(
                $this->api->url('..'));
            $this->api->template->trySet('page_title',
                sprintf('%s is looking for %s',$m->get('company'),$m->get('position')));
        }
        function defaultTemplate(){
            return array('page/jobs');
        }
    }
    // Only used on this page 
    class JobList extends Grid {   // added this class
        function format_link($field){

            $parts=array(
                $this->current_row['location'],
                $this->current_row['company'],
                $this->current_row['id'],
                $this->current_row['position'],
                );

            $parts=preg_replace('/[^a-zA-Z 0-9-]/','',$parts);
            $parts=preg_replace('/^$/','-',$parts);
            $parts=str_replace(' ','_',$parts);
            $page=implode('/',$parts);

            $this->current_row[$field]='<a href="'.
                    $this->api->url('job/'.$page).
                    '">'.$this->current_row[$field].'</a>';
        }
    }
If you are thinking, why this sort of thing is not handled by the framework, the answer is Agile Toolkit provides a simplest way to handle things. It is here to simplify development not overcomplicate it. There might be an add-on however which does exactly what you need.

### Why not let Model handle this
The general rule is that Models handle business logic. URL is a presentation logic and must be handled by the UI code. You shouldn't be putting things like URL generation into the Model. However you can introduce helper or define method inside the API class which makes those URLs for you.

## Page URL Generation
----
Agile Toolkit is being quite intelligent about generating relative URLs. Agile Toolkit still recognizes our details page like that, behind all the mirrors of mod_rewrite. That's why when url('..') is called from jobs_details it properly points to page "jobs". There are more ways how to define relative URLs such as if you would want to make link from day5 to day6 on this guide, you would write <?page?>../day6<?page?>. If you are looking to find link to subpage, you can use './subpage'.

### SEE YOU TOMORROW

This was surprisingly quick lesion, because Agile Toolkit did not introduce complex things, but instead focuses on simplifying things which are already complex.

Tomorrow, we won't introduce any new concepts, but rather spend time going deeper into what we've covered so far.