How to upgrade to 1.1
=====================


##General

The Factory Girl has been significantly improved with the great contributions of [@franmomu][franmomu]. If you don't use factories in your functional tests (you should), you can skip this reading.


##The old convention approach

In the 1.0 version of the FactoryGirl, many decisions were made based on conventions. In order to create a factory for an entity, you passed the name of the entity as an argument to build/create and FactoryGirl then looked for a class named "Entity" + "Factory" in the defined namespace. This approach had many flaws:

- All your factories had to be in the same namespace. This made difficult to decouple bundles and share them between projects.

- You had no control over dependency injection. It was the FactoryGirl itself who handled it. So if you needed other services in your factories, you had to create them in constructor or pass them to build/create method.


##The new configuration approach
The new approach needs a bit more of configuration stuff, but adds a lot of flexibility. It uses a [Compiler Pass][compiler_pass] to collect all factories and pass them to FactoryGirl. To do that, you need to register your factories as services and tag them properly:


    services:
        your_vendor.handy_test.person_factory:
            class: Path\To\Your\Factory
            arguments: ["@doctrine.orm.entity_manager"]
            tags:
                - { name: handy_tests.factory }


After that, you have to implement the new method getName() added to FactoryInterface:


    namespace Your\OwnBundle\Factory;

    use Doctrine\Common\Persistence\ObjectManager;
    use BladeTester\HandyTestsBundle\Model\FactoryInterface;

    use Your\OwnBundle\Entity\Person;

    class PersonFactory implements FactoryInterface {

        private $om;


        // ...


        public function getName()
        {
            return 'Person';
        }
    }


Don't forget to delete the old definition in your config_test.yml:

    services:
        handy_tests.factory_girl:
            class: "BladeTester\HandyTestsBundle\Model\FactoryGirl"
            arguments: ['Your\MainBundle\Tests\Factory', "@doctrine.orm.entity_manager"]


And that's it! If you have factories of different bundles, now you can define the proper namespace and move them. Also, if you have other dependencies (i.e. one factory that needs another) you can easily add them to the constructor arguments.



[franmomu]: https://github.com/franmomu
[compiler_pass]: http://symfony.com/doc/current/cookbook/service_container/compiler_passes.html
