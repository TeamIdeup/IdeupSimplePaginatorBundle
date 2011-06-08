# SimplePaginator bundle for Symfony2/Doctrine2

This package contains a bundle to easily paginate complex queries efficiently and without effort.

## Including SimplePaginatorBundle in your code

You should clone this repository in your Symfony's `vendor/bundles` directory, add it into `autoload.php` file:

    
    $loader->registerNamespaces(array(
      'Symfony' => array(__DIR__.'/../vendor/symfony/src', __DIR__.'/../vendor/bundles'),
      ...
      'Ideup'   => __DIR__.'/../vendor/bundles',
      );

... and in your `AppKernel.php` file:

    public function registerBundles()
    {
        $bundles = array(
          ...
            new Ideup\SimplePaginatorBundle\IdeupSimplePaginatorBundle(),
          );

... so you are ready now to use IdeupSimplePaginatorBundle as a service.

Since the `Paginator::paginate()` method needs a `Query` object to work with, you need to change slightly your entity Repository classes:

  * Before

      class User extends EntityRepository 
      {
        public function findByMyCriteria() {
          $query = $this->_em->createQuery('...');
          return $query->getResult();
        }
      }

  * After

      class User extends EntityRepository 
      {
        public function findByMyCriteria() {
          return $this->findByMyCriteriaDQL()->getResult();
        }

        public function findByMyCriteriaDQL() {
          $query = $this->_em->createQuery('...');
          return $query;
        }
      }

In your controller you can be able to instantiate the paginator service. SimplePaginatorBundle is smart enough to
detect the current page and the maximum items per page from the Request context, so you don't need to type more 
boilerplate code!

    class MyController extends Controller
    {
      public function listAction() {
        $paginator = $this->get('ideup.simple_paginator');

        $users = $paginator->paginate($em->getRepository('MyBundle:User')->findByMyCriteriaDQL())->getResult();

        $vars = array(
            'users'     => $users,
            'paginator' => $paginator);
        return $this->render('MyBundle:User:list.html.twig', $vars);
      }
    }

Note that the variable `$users` contains only the paginated subset of the Doctrine collection and you can query
`$paginator` object to obtain information about the pagination process; such as how many items are in the full
collection, in wich page are we, wich is the last page, etc.


Authors:
 - javiacei
 - cordoval
 - Moisés Maciá

Changelog:
 - Added dependency to DoctrineExtensions\Paginate to handle proper pagination (see https://github.com/beberlei/DoctrineExtensions)
 - Added support to paginate multiple lists at once
 - Changed the Paginador class name to Paginator
   this is how services.xml defines our service, with a parameter set to the class implementing pagination and passing a service id
 
       <parameters>
           <parameter key="simple_paginador.class">Ideup\SimplePaginatorBundle\Paginator\Paginator</parameter>
       </parameters>
 
       <services>
           <service id="ideup.simple_paginator" class="%simple_paginator.class%">
               <argument type="service" id="request" strict="false" />
           </service>
       </services>
 
 - implemented Twig extension and calling the Class Paginator from there

TODO:
  1 - Testing it with a hello world sample
  2 - Making it work with a more generic Doctrine Collections

