# hoclaravel8x
https://webmobtuts.com/backend-development/using-imap-pop3-with-laravel-to-fetch-mailbox/
https://vinasupport.com/su-dung-thu-vien-simple-html-dom-voi-laravel/
Using Imap/POP3 With Laravel To Fetch Mailbox
SHARE
using IMAP and POP3 with laravel
Wael SalahBy
Wael Salah
Share
Have you ever looking for away to read your personal mailbox such as Gmail or Yahoo, in this article i will show how to do this in php and laravel.


 


 
 

Many enterprise based systems such as Customer Relationship Managements (CRM) contain components for managing mailboxes and emails from inside the system itself providing the system admins to monitor their mailbox, compose email messages without going to email clients such as outlook.

PHP supports connecting and reading mailboxes using the IMAP or POP3 protocols, and in this post i will demonstrate this using laravel framework. We will use a laravel package Webklex/laravel-imap it’s based on PHP imap extension.

 

Installation
First you need to be sure that php imap extension library is installed otherwise install it using:

sudo apt-get install php*-imap php*-mbstring php*-mcrypt && sudo apache2ctl graceful
After installing add it to your php.ini restart your apache server. Also check your phpinfo() if it contains imap then it was installed successfully.

Next install Webklex/laravel-imap using composer in your laravel project using this command:

composer require webklex/laravel-imap
If you’re running Laravel >= 5.5, package discovery will configure the service provider and Client alias out of the box.

Otherwise, for Laravel <= 5.4, edit your config/app.php as follows:

add the following to the providers array: 
Webklex\IMAP\Providers\LaravelServiceProvider
add the following to the aliases array:
Client=>Webklex\IMAP\Facades\Client::class
After that run this command to generate the config file:

php artisan vendor:publish --provider="Webklex\IMAP\Providers\LaravelServiceProvider"
 


 
Configuration
1- Single Account
If you intend to use the package for only a single account then the process is pretty simple, add those configuration items into the .env file:

IMAP_HOST=somehost.com
IMAP_PORT=993
IMAP_ENCRYPTION=ssl
IMAP_VALIDATE_CERT=true
IMAP_USERNAME=root@example.com
IMAP_PASSWORD=secret
IMAP_DEFAULT_ACCOUNT=default
IMAP_PROTOCOL=imap
1- Multi Accounts
If you intend to use it for multi accounts such as every user in the system can control their mailbox it’s preferred to store the above variables into the database in a separate table connected with user_id.

 

If you open config/imap.php their will a lot of options:

<?php
return [
   
 'default' => env('IMAP_DEFAULT_ACCOUNT', 'default'),
'accounts' => [
        'default' => [// account identifier
            'host'  => env('IMAP_HOST', 'localhost'),
            'port'  => env('IMAP_PORT', 993),
            'protocol'  => env('IMAP_PROTOCOL', 'imap'), //might also use imap, [pop3 or nntp (untested)]
            'encryption'    => env('IMAP_ENCRYPTION', 'ssl'), // Supported: false, 'ssl', 'tls'
            'validate_cert' => env('IMAP_VALIDATE_CERT', true),
            'username' => env('IMAP_USERNAME', 'root@example.com'),
            'password' => env('IMAP_PASSWORD', ''),
        ],
        /*
        'gmail' => [ // account identifier
            'host' => 'imap.gmail.com',
            'port' => 993,
            'encryption' => 'ssl', // Supported: false, 'ssl', 'tls'
            'validate_cert' => true,
            'username' => 'example@gmail.com',
            'password' => 'PASSWORD',
        ],
        'another' => [ // account identifier
            'host' => '',
            'port' => 993,
            'encryption' => false, // Supported: false, 'ssl', 'tls'
            'validate_cert' => true,
            'username' => '',
            'password' => '',
        ]
        */
    ],
    'options' => [
        'delimiter' => '/',
        'fetch' => \Webklex\IMAP\IMAP::FT_UID,
        'fetch_body' => true,
        'fetch_attachment' => true,
        'fetch_flags' => true,
        'message_key' => 'id',
        'fetch_order' => 'asc',
        'open' => [
            // 'DISABLE_AUTHENTICATOR' => 'GSSAPI'
        ],
        'decoder' => [
            'message' => [
                'subject' => 'utf-8' // mimeheader
            ],
            'attachment' => [
                'name' => 'utf-8' // mimeheader
            ]
        ]
    ],
    
    'masks' => [
        'message' => \Webklex\IMAP\Support\Masks\MessageMask::class,
        'attachment' => \Webklex\IMAP\Support\Masks\AttachmentMask::class
    ]
];
 

default: This is the default account
accounts: This array contains all available accounts such as default and gmail. You can add as many accounts.
delimiter — you can use any supported char such as “.”, “/”, etc
fetch — IMAP::FT_UID (message marked as read by fetching the message) or IMAP::FT_PEEK (fetch the message without setting the “read” flag)
fetch_body — If set to false all messages will be fetched without the body and any potential attachments
fetch_attachment — If set to false all messages will be fetched without any attachments
fetch_flags — If set to false all messages will be fetched without any flags
message_key — Message key identifier option useful when you want to save the messages into the database.
fetch_order — Message fetch order by ascending or descending
open — special configuration for imap_open()
DISABLE_AUTHENTICATOR — Disable authentication properties.
decoder — Currently only the message subject and attachment name decoder can be set
masks — Default masking config
message — Default message mask
attachment — Default attachment mask
 

Example Fetching all messages
use Webklex\IMAP\Client;
$oClient = new Client([
    'host'          => 'somehost.com',
    'port'          => 993,
    'encryption'    => 'ssl',
    'validate_cert' => true,
    'username'      => 'username',
    'password'      => 'password',
    'protocol'      => 'imap'
]);
//Connect to the IMAP Server
$oClient->connect();
//Get all Mailboxes
/** @var \Webklex\IMAP\Support\FolderCollection $aFolder */
$aFolder = $oClient->getFolders();
//Loop through every Mailbox
/** @var \Webklex\IMAP\Folder $oFolder */
foreach($aFolder as $oFolder){
    //Get all Messages of the current Mailbox $oFolder
    /** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
    $aMessage = $oFolder->messages()->all()->get();
    
    /** @var \Webklex\IMAP\Message $oMessage */
    foreach($aMessage as $oMessage){
        echo $oMessage->getSubject().'<br />';
        echo 'Attachments: '.$oMessage->getAttachments()->count().'<br />';
        echo $oMessage->getHTMLBody(true);
    }
}
Sometimes when you run this example it takes a long time to execute and this is because this code fetch all messages in your inbox and this can be a huge number so the best way is to limit the results as demonstrated below in the pagination and limit sections.

 

Using the Facade

use Webklex\IMAP\Facades\Client;
$oClient = Client::account('default');
$oClient->connect();
Get specific folder by name:
$oFolder = $oClient->getFolder('INBOX.read');  // get the read inbox

 
Searching For Messages:
//Get all messages
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->all()->get();
//Get all messages from example@domain.com
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->from('example@domain.com')->get();
//Get all messages since march 15 2018
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->since('15.03.2018')->get();
//Get all messages within the last 5 days
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->since(now()->subDays(5))->get();
//Or for older laravel versions..
$aMessage = $oFolder->query()->since(\Carbon::now()->subDays(5))->get();
//Get all messages containing "hello world"
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->text('hello world')->get();
//Get all unseen messages containing "hello world"
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->unseen()->text('hello world')->get();
//Extended custom search query for all messages containing "hello world" 
//and have been received since march 15 2018
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()->text('hello world')->since('15.03.2018')->get();
$aMessage = $oFolder->query()->Text('hello world')->Since('15.03.2018')->get();
$aMessage = $oFolder->query()->whereText('hello world')->whereSince('15.03.2018')->get();
// Build a custom search query
/** @var \Webklex\IMAP\Support\MessageCollection $aMessage */
$aMessage = $oFolder->query()
->where([['TEXT', 'Hello world'], ['SINCE', \Carbon::parse('15.03.2018')]])
->get();
Refer to Webklex/laravel-imap for more options on searching.

 

Result limiting
To limit the results you can use function limit(number, page):

//Get all messages for page 2 since march 15 2018 where each page contains 10 messages
$aMessage = $oFolder->query()->limit(10, 2)->get();
 

Pagination
Use function paginate() to paginate:

$paginator = $oFolder->query()->paginate();
Paginate a message collection:

$paginator = $aMessage->paginate();
By default the pagination displays 15 results per page but you can customize it like this:

$paginator = $oFolder->search()
->since(\Carbon::now()->subDays(14))->get()
->paginate(5);
<table>
    <thead>
        <tr>
            <th>UID</th>
            <th>Subject</th>
            <th>From</th>
            <th>Attachments</th>
        </tr>
    </thead>
    <tbody>
        @if($paginator->count() > 0)
            @foreach($paginator as $oMessage)
                <tr>
                    <td>{{$oMessage->getUid()}}</td>
                    <td>{{$oMessage->getSubject()}}</td>
                    <td>{{$oMessage->getFrom()[0]->mail}}</td>
                    <td>{{$oMessage->getAttachments()->count() > 0 ? 'yes' : 'no'}}</td>
                </tr>
            @endforeach
        @else
            <tr>
                <td colspan="4">No messages found</td>
            </tr>
        @endif
    </tbody>
</table>
{{$paginator->links()}}
 

Messages
Get specific message
Get a specific message by uid (Please note that the uid is not unique and can change):

$oMessage = $oFolder->getMessage($uid = 1);
Set message flags:
$oMessage->setFlag(['Seen', 'Spam']);
$oMessage->unsetFlag('Spam');

// Mark all messages as "read" while fetching:
$aMessage = $oFolder->query()->text('Hello world')->markAsRead()->get();
// Don't mark all messages as "read" while fetching:
$aMessage = $oFolder->query()->text('Hello world')->leaveUnread()->get();

 
Attachments
Get message attachments
// Save message attachments:
$aAttachment = $oMessage->getAttachments();
$aAttachment->each(function ($oAttachment) {
    /** @var \Webklex\IMAP\Attachment $oAttachment */
    $oAttachment->save();
});
 

Fetch messages without body fetching (decrease load) useful in listing pages:
$aMessage = $oFolder->query()->whereText('Hello world')->setFetchBody(false)->get();
$aMessage = $oFolder->query()->whereAll()->setFetchBody(false)->setFetchAttachment();
This example will disable body fetching to speed the retrieval process while fetching.

 

Fetch messages without body, flag and attachment fetching (decrease load) useful in listing pages:
$aMessage = $oFolder->query()->whereText('Hello world')
->setFetchFlags(false)
->setFetchBody(false)
->setFetchAttachment(false)
->get();
$aMessage = $oFolder->query()->whereAll()
->setFetchFlags(false)
->setFetchBody(false)
->setFetchAttachment(false)
->get();
This example will disable body and attachment fetching to speed the retrieval process while fetching.

 

Best practices
When you intend to read a mailbox in your own system try to save the messages into the database.
Use a cronjob to read and save the messages in the background.
When saving the data check if the message not exist in the database otherwise save it.
Always save the important data only such as subject, body, from, to, cc, bcc, and attachments.
Create a sync button which syncs your local messages with the remote messages.
