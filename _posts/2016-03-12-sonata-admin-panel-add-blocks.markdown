---
layout: post
title:  "Add custom blocks to SonataAdminBundle admin panel in Symfony"
date:   2016-03-12 17:07:04 +0100
categories: development, symfony, sonata, php
---
Adding custom blocks to your admin panel is not as straightforward as you would
expect.
Here are the basics to do so :

# 1) Extend BaseBlockService
I created the file *TEN/AdminBundle/Block/Service/StatisticsBlockService.php* with
the following content :

{% highlight php %}
<?php

namespace TEN\AdminBundle\Block\Service;

use Symfony\Component\HttpFoundation\Response;
use Sonata\AdminBundle\Form\FormMapper;
use Sonata\AdminBundle\Validator\ErrorElement;
use Sonata\BlockBundle\Model\BlockInterface;
use Sonata\BlockBundle\Block\BaseBlockService;
use Sonata\BlockBundle\Block\BlockContextInterface;
use Symfony\Component\OptionsResolver\OptionsResolverInterface;
use Doctrine\ORM\EntityManager;

class StatisticsBlockService extends BaseBlockService
{
    private $em;

    public function setDefaultSettings(OptionsResolverInterface $resolver)
    {
        $resolver->setDefaults(array(
            'template' => 'TENAdminBundle:Block:block_statistics.html.twig',
        ));
    }

    public function execute(BlockContextInterface $blockContext, Response $response = null)
    {
        // merge settings
        $settings = $blockContext->getSettings();

        $userRepository = $this->em->getRepository('TENUserBundle:User');
        $activatedUsers = $userRepository->countActivated();

        return $this->renderResponse($blockContext->getTemplate(), array(
            'activatedUsers' => $activatedUsers,
            'block'     => $blockContext->getBlock(),
            'settings'  => $settings,
        ), $response);
    }

    public function __construct($name, $templating, EntityManager $entityManager)
    {
            parent::__construct($name, $templating);
            $this->em = $entityManager;
    }
}
{% endhighlight %}

# 2) Create the twig file for your custom block

I created the file *TEN/AdminBundle/Resources/views/Block/block_statistics.html.twig*
with the following content :

{% highlight twig %}{% raw %}
{% extends sonata_block.templates.block_base %}

{% block block %}
    <div class="col-lg-3 col-xs-6">
        <div class="small-box bg-green">
            <div class="inner">
                <h3>{{ activatedUsers }}</h3>

                <p>Activated</p>
            </div>
            <div class="icon">
                <i class="fa fa-users"></i>
            </div>
            <a href="ten/user/user/list" class="small-box-footer">
                View more <i class="fa fa-arrow-circle-right"></i>
            </a>
        </div>
    </div>
{% endblock %}
{% endraw %}{% endhighlight %}

# 3) Register the new service StatisticsBlockService

{% highlight yalm %}
sonata.block.service.statistics:
        class: TEN\AdminBundle\Block\Service\StatisticsBlockService
        tags:
            - name: sonata.block
              manager_type: orm
              label: Statistics
        arguments:
            - "sonata.block.service.statistics"
            - "@templating"
            - "@doctrine.orm.entity_manager"
{% endhighlight %}

# 4) Edit config.yml

Finally add the following lines to *config.yml* :

{% highlight yalm %}
sonata_block:
    blocks:
        sonata.block.service.statistics: ~

sonata_admin:
    dashboard:
        blocks:
            -
                position: top
                class: col-md-12
                type: sonata.block.service.statistics
{% endhighlight %}

That's it!
