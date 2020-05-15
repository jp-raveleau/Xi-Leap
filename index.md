<html>
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <title>DR with AHV Sync Rep and Leap | Nutanix&trade; Workshops</title>
        <meta name="robots" content="noindex, nofollow">
        <link rel="stylesheet" media="all" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" media="all" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/font-awesome/css/font-awesome.min.css">
        <link rel="stylesheet" media="screen" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/workshops/css/screen.css">
        <link rel="stylesheet" media="print" href="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/workshops/css/print.css">

        <link rel="shortcut icon" href="https://s3.amazonaws.com/handsonworkshops.prod.media/a/a/logo/nutanix-favicon.ico">

    </head>

<span id="xileap"></span><h1>vExpert Session - Frictionless DR with Xi Leap<a class="headerlink" href="#frictionless-dr-with-xi-leap" title="Permalink to this headline">¶</a></h1>
<li>
</li>
<p>Legacy disaster recovery configurations, which are created with Prism Element, use protection domains and third-party integrations to protect VMs, and they replicate data between on- premises Nutanix clusters. Protection domains provide limited flexibility in terms of supporting operations such as VM boot order and require you to perform manual tasks to protect new VMs as an application scales up.</p>
<p>Leap uses an entity-centric approach and runbook-like automation to recover applications. It uses categories to group the entities to be protected and to automate the protection of new entities as the application scales. Application recovery is more flexible with network mappings, configurable stages to enforce a boot order, and optional inter-stage delays. Application recovery can also be validated and tested without affecting production workloads. All the configuration information that an application requires upon failover are synchronized to the recovery location.</p>
<p>You can use Leap between two physical data centers or between a physical data center and Xi Cloud Services. Leap works with pairs of physically isolated locations called availability zones. One availability zone serves as the primary location for an application while a paired availability zone serves as the recovery location. While the primary availability zone is an on-premises Prism Central instance, the recovery availability zone can be either on-premises or in Xi Cloud Services.</p>
<p><strong>In this lab you will explore Xi Leap, configuring a Protection Policy, building a Recovery Plan, evaluating networking considerations, and perform a failover.</strong></p>
<div class="section" id="accessing-your-lab-environment">
<span id="accessingleaplab"></span><h2>Accessing Your Lab Environment<a class="headerlink" href="#accessing-your-lab-environment" title="Permalink to this headline">¶</a></h2>
<p>This lab requires both a Nutanix Xi Leap account and a traditional on-premises Nutanix cluster. In this brief exercise you will retrieve the the connection and credential details that have been reserved for you.</p>
<ol class="arabic">
<li><p class="first">Obtain your unique Xi Leap Test Drive account.</p>
</li>
<li><p class="first">Open <a class="reference external" href="https://testdrive.leap.nutanix.com">https://testdrive.leap.nutanix.com</a> in your browser. The VPN is required.</p>
</li>
<li><p class="first">Provide your <strong>contact informations</strong> and click <strong>Request Test Drive</strong>. Wait for the Nutanix e-mail with all credentials (look in your spam folder if necessary)</p>
<div class="figure">
<img alt="../../_images/TestDrive.png" src="../../_images/TestDrive.png" />
</div>
</li>
<div class="figure">
<img alt="../../_images/mailreceived.png" src="../../_images/mailreceived.png" />
</div>
<li>
</li>
<li><p class="first">Under <strong>On-Prem</strong>, click <strong>Launch</strong> to access Prism Central for your on-premises Nutanix cluster.</p>
</li>
<li><p class="first">Log in using the displayed <strong>Username</strong> and <strong>Password</strong>.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">If prompted, ignore the SSL Certificate warning and proceed to the URL.</p>
</div>
</li>
<li><p class="first">Under <strong>Xi-Cloud</strong>, click <strong>Launch</strong> and again login in with the corresponding <strong>Username</strong> and <strong>Password</strong></p>
</li>
</ol>
</div>
<div class="section" id="verifying-cluster-pairing">
<h2>Verifying Cluster Pairing<a class="headerlink" href="#verifying-cluster-pairing" title="Permalink to this headline">¶</a></h2>
<p><strong>Availability Zones</strong> represent physically separate groups of resources, whether that be Nutanix clusters across different sites, or Xi Cloud Services environments. In order to replicate between Availability Zones, your source cluster must first be paired to the target Prism Central or Xi environment.</p>
<ol class="arabic">
<li><p class="first">In your <strong>On-Prem Prism Central</strong>, click <em class="fa fa-bars"></em> <strong>&gt; Administration &gt; Availability Zones</strong>.</p>
<div class="figure">
<img alt="../../_images/340.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/340.png" />
</div>
</li>
<li><p class="first">Note that your <strong>On-Prem</strong> cluster has already been paired with the <strong>Xi-US-WEST</strong> cloud environment.</p>
</li>
<li><p class="first">Optionally, click <strong>Connect to Availability Zone</strong> to see what details are required to pair a new cluster or Xi Cloud environment.</p>
</li>
</ol>
</div>
<div class="section" id="creating-categories">
<h2>Creating Categories<a class="headerlink" href="#creating-categories" title="Permalink to this headline">¶</a></h2>
<p>In <strong>Prism Central</strong>, a <strong>Category</strong> is a key value pair. Categories are assigned to entities (such as VMs, Networks, or Images) based on some criteria (Location, Production-level, App Name, etc.). Different policies can then be mapped to categories, applying to all VMs with the appropriate assigned values.</p>
<p>For example, you might have a Department category that includes values such as Engineering, Finance, and HR. In this case you could create one backup policy that applies to Engineering and HR and a separate (more stringent) backup policy that applies to just Finance. Categories allow you to implement a variety of policies across entity groups, and Prism Central allows you to quickly view any established relationships.</p>
<ol class="arabic">
<li><p class="first">In your <strong>On-Prem Prism Central</strong>, click <em class="fa fa-bars"></em> <strong>&gt; Virtual Infrastructure &gt; Categories</strong>.</p>
<div class="figure">
<img alt="../../_images/424.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/424.png" />
</div>
</li>
<li><p class="first">Select the <strong>Demo</strong> category and click <strong>Actions &gt; Update</strong>.</p>
<div class="figure">
<img alt="../../_images/524.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/524.png" />
</div>
</li>
<li><p class="first">Observe the category already has 3 available values. Click the <em class="fa fa-plus-circle"></em> icon beside the last value to add <strong>App-4</strong>. Click <strong>Save</strong>.</p>
<div class="figure">
<img alt="../../_images/623.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/623.png" />
</div>
<p>Now you’ll need to assign your category to an entity.</p>
</li>
<li><p class="first">Click <em class="fa fa-bars"></em> <strong>&gt; Virtual Infrastructure &gt; VMs</strong>.</p>
</li>
<li><p class="first">Take note of the IP addresses for the deployed VMs.</p>
</li>
<li><p class="first">Select the <strong>App-4</strong> VM and click <strong>Actions &gt; Manage Categories</strong>.</p>
</li>
<li><p class="first">In the search field, specify <strong>Demo: App-4</strong> and click <strong>Save</strong> to apply the category to the VM.</p>
<div class="figure">
<img alt="../../_images/722.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/722.png" />
</div>
</li>
</ol>
</div>
<div class="section" id="updating-protection-policy">
<h2>Updating Protection Policy<a class="headerlink" href="#updating-protection-policy" title="Permalink to this headline">¶</a></h2>
<p>A <strong>Protection Policy</strong> defines the desired RPO and snapshot policy.</p>
<ol class="arabic">
<li><p class="first">In your <strong>On-Prem Prism Central</strong>, click <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Protection Policies</strong>.</p>
</li>
<li><p class="first">Select the existing policy, <strong>AppPP</strong>, and click <strong>Actions &gt; Update</strong>.</p>
<div class="figure">
<img alt="../../_images/822.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/822.png" />
</div>
</li>
<li><p class="first">Observe the different options for remote and local snapshot retention:</p>
<ul>
<li><p class="first"><strong>Linear</strong></p>
<blockquote>
<div><p>Implements a simple, linear retention scheme at both local and remote sites. If you set the retention number for a given site to n, the n most recent snapshots are retained at that site. For example, if the RPO is one hour and theretention number for the local site is 48, the 48 most recent snapshots are retained at any given time.</p>
</div></blockquote>
</li>
<li><p class="first"><strong>Roll-up</strong></p>
<blockquote>
<div><p>Rolls up the oldest snapshots for the specified RPO interval into a single snapshot when the next higher interval is reached, all the way up to the retention period specified for a site. For example, if you select roll-up retention, set the RPO to one hour, and set the retention time at a site to one year, the twenty four oldest hourly backups at that site are rolled up into a single daily backup at the completion of every 24 hours, the seven oldest daily backups are rolled up into a single weekly backup at the completion of every week, the four oldest weekly backups are rolled up into a single backup at the completion of every month, and the twelve oldest monthly backups are rolled up into a single backup at the completion of every year. At the end of one year, that site has 24 of the most recent hourly backups, seven of the most recent daily backups, four of the most recent weekly backups, twelve of the most recent monthly backups, and one yearly backup. The snapshots that are used to create a rolled-up snapshot are discarded.</p>
</div></blockquote>
</li>
</ul>
<p>Next, you’ll include your new category in the Protection Policy, ensuring all <strong>App-4</strong> tagged VMs share this policy.</p>
</li>
<li><p class="first">Click <strong>Update Categories</strong> and specify <strong>Demo: App-4</strong> in the <strong>Add Categories</strong> dialog.</p>
<div class="figure">
<img alt="../../_images/923.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/923.png" />
</div>
</li>
<li><p class="first">Click <strong>Save</strong>.</p>
</li>
</ol>
</div>
<div class="section" id="updating-recovery-plan">
<h2>Updating Recovery Plan<a class="headerlink" href="#updating-recovery-plan" title="Permalink to this headline">¶</a></h2>
<p>A <strong>Recovery Plan</strong> defines the runbook for a failover event, including power on sequences and network mappings.
]
#. In your <strong>On-Prem Prism Central</strong>, click <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Recovery Plan</strong>.</p>
<ol class="arabic">
<li><p class="first">Select the existing policy, <strong>AppRP</strong>, and click <strong>Actions &gt; Update</strong>.</p>
<div class="figure">
<img alt="../../_images/1022.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1022.png" />
</div>
</li>
<li><p class="first">Click <strong>Next</strong>. Observe the existing stages for powering on VMs as part of the recovery plan.</p>
</li>
<li><p class="first">Under <strong>Power On Sequence</strong>, click <strong>Add New Stage</strong> to add a 4th stage.</p>
<div class="figure">
<img alt="../../_images/1124.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1124.png" />
</div>
</li>
<li><p class="first">To add a delay between two stages, click <strong>Add Delay</strong> between the two stages and specify a value in <strong>Seconds</strong>. Click <strong>Add</strong>.</p>
</li>
<li><p class="first">Under the <strong>Actions</strong> menu for your new stage, add the <strong>Demo: App-4</strong> category.</p>
</li>
<li><p class="first">Click <strong>Next</strong>.</p>
<p><strong>Network Settings</strong> enables you to map networks in the local availability zone (the primary location) to networks at the recovery location. When failover occurs and VMs are recovered at the recovery location, they are placed in the network that is mapped to their network on the primary location.</p>
<div class="figure">
<img alt="../../_images/1223.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1223.png" />
</div>
<p>Local availability zone on the left. On the left, you specify the VM Networks used for both <strong>Production</strong> and <strong>Test</strong> failover events. These are mapped to corresponding <strong>Production</strong> and <strong>Test</strong> networks in Xi Cloud. If the recovery location is an on-premises availability zone, you would then specify the corresponding VM Networks for that site.</p>
<p>Optionally, you can specify a gateway IP address and prefix. With Xi Cloud Services, you either select a subnet that you have created, or you can enter the gateway IP address and prefix length. If you specify a gateway IP address and prefix length, the recovery plan dynamically creates the subnet on failover, and it cleans up the dynamically created subnets after failback. This functionality is a key benefit of DR with Xi Leap, as each VM will be able to maintain its original IP address when failing over to Xi Cloud, drastically reducing runbook scripting to re-configure applications.</p>
<p>Observe that one VM has been assigned a <strong>Floating IP</strong>. This will provide direct access to the VM from the public Internet. Upon failover you will observe the public IP that gets assigned to the VM in a NAT configuration.</p>
</li>
</ol>
</div>
<div class="section" id="exploring-xi-cloud-portal">
<h2>Exploring Xi Cloud Portal<a class="headerlink" href="#exploring-xi-cloud-portal" title="Permalink to this headline">¶</a></h2>
<ol class="arabic">
<li><p class="first">Return to your Xi Cloud session using the <strong>Launch</strong> link and credentials provided in <a class="reference internal" href="#accessingleaplab"><span class="std std-ref">Accessing Your Lab Environment</span></a>.</p>
</li>
<li><p class="first">Take some time to explore the Xi Cloud interface, comparing it to your on-premises Prism Central experience.</p>
</li>
<li><p class="first">Click <strong>Explore &gt; Virtual Private Clouds</strong>.</p>
</li>
<li><p class="first">Select the <strong>Production</strong> network.</p>
<p>This is where you can create up to 100 different subnets and build policy-based routing rules between them.</p>
</li>
<li><p class="first">Select the <strong>VPN</strong> tab.</p>
<div class="figure">
<img alt="../../_images/1323.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1323.png" />
</div>
<p>While networking can be the biggest challenge of any hybrid cloud deployment, Nutanix strives to simplify this process (and reduce unnecessary professional services costs) by providing automatic configuration options. Stepping through the automatic configuration wizard will also provide all of the commands required to complete the configuration of your on-premises environment.</p>
</li>
</ol>
</div>
<div class="section" id="performing-a-failover">
<h2>Performing A Failover<a class="headerlink" href="#performing-a-failover" title="Permalink to this headline">¶</a></h2>
<p>Leap supports the following types of failover operations:</p>
<ul>
<li><p class="first"><strong>Test Failover</strong></p>
<blockquote>
<div><p>You perform a test failover when you want to test a recovery plan. When you perform a test failover, the VMs are started in the virtual network designated for testing purposes at the recovery location (a manually created virtual network on on-premises clusters and a virtual subnet in the Test VPC in Xi Cloud Services). However, the VMs at the primary location are not affected. Test failovers rely on the presence of VM snapshots at the recovery location.</p>
</div></blockquote>
</li>
<li><p class="first"><strong>Planned Failover</strong></p>
<blockquote>
<div><p>You perform planned failover when a disaster that disrupts services is predicted at the primary location. When you perform a planned failover, the recovery plan first creates a snapshot of each VM, replicates the snapshots at the recovery location, and then starts the VMs at the recovery location. Therefore, for a planned failover to succeed, the VMs must be available at the primary location. If the failover process encounters errors, you can resolve the error condition. After a planned failover, the VMs no longer run in the source availability zone.</p>
<p>After failover, replication begins in the reverse direction. For a planned failover the MAC address will be maintained.</p>
</div></blockquote>
</li>
<li><p class="first"><strong>Unplanned Failover</strong></p>
<blockquote>
<div><p>Hide ya kids, hide ya wife. You perform unplanned failover when a disaster has occurred at the primary location. In an unplanned failover, you can expect some data loss to occur. The maximum data loss possible is equal to the RPO configured in the protection policy or the data that was generated after the last manual backup for a given VM. In an unplanned failover, by default, VMs are recovered from the most recent snapshot. However, you can recover from an earlier snapshot by selecting a date and time. Any errors are logged but the execution of the failover continues.</p>
<p>After failover, replication begins in the reverse direction.</p>
<div class="admonition note">
<p class="first admonition-title">Note</p>
<p class="last">You can perform an unplanned failover operation only if snapshots have been replicated to the recovery availability zone. At the recovery location, failover operations cannot use snapshots that were created locally in the past. For example, if you perform a planned failover from the primary availability zone AZ1 to recovery location AZ2 (Xi Cloud Services) and then attempt an unplanned failover from AZ2 to AZ1, recovery will succeed at AZ1 only if snapshots were replicated from AZ2 to AZ1 after the planned failover operation. The unplanned failover operation cannot perform recovery based on snapshots that were created locally when the entities were running in AZ1.</p>
</div>
</div></blockquote>
</li>
</ul>
<ol class="arabic">
<li><p class="first">From <strong>Xi Cloud</strong>, select <strong>Explore &gt; Recovery Plans</strong>.</p>
</li>
<li><p class="first">Select your <strong>AppRP</strong> plan and click <strong>Actions &gt; Failover</strong>.</p>
<div class="figure">
<img alt="../../_images/1422.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1422.png" />
</div>
</li>
<li><p class="first">Note the number of entities to be failed over as part of the plan. Click <strong>Failover</strong>.</p>
<div class="figure">
<img alt="../../_images/1519.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1519.png" />
</div>
</li>
<li><p class="first">Once the recovery plan updates to <strong>Running</strong>, click <strong>AppRP</strong> and select <strong>Tasks</strong> to view the current status. Continue to observe the failover, taking note of the different boot stages.</p>
<div class="figure">
<img alt="../../_images/1620.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1620.png" />
</div>
</li>
<li><p class="first">Once the failover operation has completed, select <strong>VMs</strong> from the sidebar.</p>
</li>
<li><p class="first">Click <strong>AppVM-1</strong> and select the <strong>NICs</strong> tab. Note the MAC and IP addresses have remained the same, and a floating, public IP has been added to the interface.</p>
<div class="figure">
<img alt="../../_images/1722.png" src="https://s3.amazonaws.com/handsonworkshops.prod.media/ws/7311424032b04999b9c942bdc02dc5bb/i/file/548fe05ba29f4f13b5e8908b4a2c2a03/1722.png" />
</div>
</li>
</ol>
</div>
<div class="section" id="failing-back">
<h2>Failing Back<a class="headerlink" href="#failing-back" title="Permalink to this headline">¶</a></h2>
<p>Not to be confused with failing up, failing back to the primary site, including the changes that took place while running at the recovery site, is a critical part of the DR workflow.</p>
<ol class="arabic simple">
<li>Return to your <strong>On-Prem Prism Central</strong> and click <em class="fa fa-bars"></em> <strong>&gt; Policies &gt; Recovery Plan</strong>.</li>
<li>Select your <strong>AppRP</strong> plan and click <strong>Actions &gt; Failover</strong>.</li>
<li>When failing back, your <strong>Recovery Location</strong> should correspond to your primary site. Click <strong>Failover</strong>.</li>
<li>Click on your recovery plan and note you have access to the same UI in Prism Central to monitor the failback operation.</li>
<li>Once the recovery plan has completed, validate the VMs are once again running.</li>
</ol>
</div>
<div class="section" id="takeaways">
<h2>Takeaways<a class="headerlink" href="#takeaways" title="Permalink to this headline">¶</a></h2>
<ul class="simple">
<li>Xi Leap delivers fast and easy cloud DR capabilities to on-premises Nutanix clusters</li>
<li>Xi Leap drastically simplifies networking requirements by providing automated VPN setup</li>
<li>Having the same networks available in Xi Cloud drastically simplify VM runbooks</li>
<li>Xi Leap supports test, planned, and unplanned failover operations</li>
<li>Failback operations from Xi Leap require no special changes to your runbooks</li>
<li>Prism Central and Xi Cloud provide a unified management experience for managing and monitoring DR operations</li>
<li>Leap can also be used to deliver native DR capabilities between multiple on-premises Nutanix AHV clusters</li>
</ul>
</div>
</div>

                    </div>
                    <div data-next-page class="next-page clearfix">
                        <a class="btn btn-primary float-right" href="#">Next: <span data-next-page-title></span> <i class="fa fa-fw fa-chevron-right btn-icon"></i></a>
                    </div>
                </div>

                </div>
            </div>
        </div>
        <footer>
            <div class="container">
                <div class="row">
                    <div class="col-sm">
                        <p>Powered by Hands-on Workshops&trade;.</p>
                    </div>
                </div>
            </div>
        </footer>

    
        <form id="impressed" class="d-none" method="post" action="/workshops/i/" enctype="multipart/form-data"><input type="hidden" name="csrfmiddlewaretoken" value="gFRSmTxNc4GmHVRpxoKDxgU3qUJFlU8UhInKAZGJwcqFEZ6P270H1DOzDUeaNsz4">
            <p><label for="id_user">user:</label> <input type="text" name="user" value="446c386ef08a4309bb7327446d3e4b0f" required id="id_user"></p>
<p><label for="id_workshop">workshop:</label> <input type="text" name="workshop" value="7311424032b04999b9c942bdc02dc5bb" required id="id_workshop"></p>
<p><label for="id_title">title:</label> <input type="text" name="title" value="Frictionless DR with Xi Leap" maxlength="255" required id="id_title"></p>
<p><label for="id_path">path:</label> <input type="text" name="path" value="xileap/xileap" maxlength="2048" required id="id_path"></p>
<p><label for="id_digest">digest:</label> <input type="text" name="digest" value="04e85c0e448f77e0337af682cabd0d11c78a691bd52669a143cd412f2ff64df75eeeec2dc557eba65ff3967832a279dc929436c577887cb1ab5a5aa4d77e1902" maxlength="128" required id="id_digest"></p>
        </form>
    
        <div data-fixed class="back-to-top">
            <a href="#top" class="btn btn-secondary"><i class="fa fa-fw fa-arrow-up"></i><span class="back-to-top-text"> Back to Top</span></a>
        </div>
    
        <div data-fixed class="call-for-help left">
            <a data-toggle="modal" data-target="#create-call" class="btn btn-warning"><i class="fa fa-fw fa-question-circle"></i></a>
        </div>
    


    

    
        <div class="modal fade" id="create-call" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-centered" role="document">
                <form id="create-call-form" class="modal-content" action="/calls/73114240-32b0-4999-b9c9-42bdc02dc5bb/create/" method="post" enctype="multipart/form-data">
                    <div class="d-none">
                        <input type="hidden" name="csrfmiddlewaretoken" value="gFRSmTxNc4GmHVRpxoKDxgU3qUJFlU8UhInKAZGJwcqFEZ6P270H1DOzDUeaNsz4">
                        <input type="hidden" name="call-document_title" value="Frictionless DR with Xi Leap" id="id_call-document_title">
                        <input type="hidden" name="call-document_path" value="xileap/xileap" id="id_call-document_path">
                    </div>
                    <div class="modal-header">
                        <h5 class="modal-title">Need help?</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <div class="form-group">
                            
<label for="id_call-location"><strong class="danger">*</strong> <strong>Location</strong></label>

<input type="text" name="call-location" maxlength="100" class="form-control" required id="id_call-location">


<small class="form-text">Let us know the room / table # / team #, etc.</small>


                        </div>
                        <div class="form-group">
                            
<label for="id_call-details">Details <small>(optional)</small></label>

<textarea name="call-details" cols="40" rows="5" maxlength="250" class="form-control" id="id_call-details">
</textarea>


<small class="form-text">Briefly describe what you need help with.</small>


                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Cancel</button>
                        <button type="submit" class="btn btn-primary">Request help</button>
                    </div>
                </form>
            </div>
        </div>
    

        <script src="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/jquery/jquery.min.js"></script>
        <script src="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/bootstrap/js/bootstrap.bundle.min.js"></script>
        <script src="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/libs/clipboard/clipboard.min.js"></script>
        <script src="https://s3.amazonaws.com/handsonworkshops.prod.static/nova/workshops/js/common.js"></script>

        <script async src="https://www.googletagmanager.com/gtag/js?id=UA-115508909-1"></script>
        <script>
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', 'UA-115508909-1', {
                'custom_map': {'dimension1': 'user_uuid', 'dimension2': 'workshop_uuid', 'dimension3': 'workshop_name'}
            });
    
            gtag('event', 'workshop_dimensions', {'workshop_uuid': '73114240-32b0-4999-b9c9-42bdc02dc5bb', 'workshop_name': 'Global Tech Summit 2020 - Americas'});
    
    
            gtag('event', 'user_uuid_dimension', {'user_uuid': '446c386e-f08a-4309-bb73-27446d3e4b0f'});
    
        </script>


    
        <script>
            window.user_data = {
                email: 'jeanpierre.raveleau@nutanix.com',
                name: 'Jean-Pierre Raveleau',
                created_at: '1522152603'
            };
        </script>
    
    
        <script>
            jQuery(function ($) {
                // Post the impression form
                $.post(
                    $('#impressed').attr('action'),
                    $('#impressed').serializeArray(),
                    function (data, status, xhr) {
                        if (typeof window.console !== 'undefined') {
                            console.log(data.success);
                        }
                    }
                );
            });
        </script>
    
    
        <script>
            jQuery(function ($) {
                // Submit create call form
                $('body').on('submit', '#create-call-form', function (event) {
                    event.preventDefault();

                    // Post the form
                    $.post(
                        $(this).attr('action'),
                        $(this).serializeArray(),
                        function (data, status, xhr) {
                            if (typeof window.console !== 'undefined') {
                                console.log(data.success);
                            }

                            if (data.success) {
                                location.reload();
                            } else if (data.errors.__all__) {
                                alert(data.errors.__all__[0]);
                            }
                        }
                    );
                });
            });
        </script>
    

    

    </body>
</html>