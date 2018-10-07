# Free Kassa component for Yii2 #

Payment gateway and api client for [Free Kassa](http://www.free-kassa.ru) service.

## Installation ##

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

Either run

    php composer.phar require --prefer-dist sdelfi/yii2-free-kassa "~1.0"

or add

    "sdelfi/yii2-free-kassa": "~1.0"

to the `require` section of your composer.json.

## Usage ##

### Component configuration ###

Configure `freeKassa` component in the `components` section of your application.

    'freeKassa' => [
        'class' => '\sdelfi\freekassa\Merchant',
        'merchantId' => 'YOUR_MERCHANT_ID',
        'merchantFormSecret' => 'SECRET_1',
        'checkDataSecret' => 'SECRET_2',
        'defaultCurrency' => 'DEFAULT_CURRENCY_ID',
        'defaultLanguage' => 'ru' // or 'en' 
    ],
    
### Redirecting to the payment system ###

To redirect user to PerfectMoney site you need to create the page with RedirectForm widget.
User will redirected right after page load.

    <?= \sdelfi\freekassa\RedirectForm::widget([
        'message' => 'Redirecting to payment gateway...',
        'api' => Yii::$app->get('freeKassa'),
        'invoiceId' => $invoice->id,
        'amount' => $invoice->amount,
        'description' => $invoice->description,
        'email' => $invoice->owner->email
    ]); ?>

### Gateway controller ###

You will need to create controller that will handle result requests from PerfectMoney service.
Sample controller code:

    <?php
    namespace frontend\controllers;
    
    use common\models\Invoice;
    use yii\base\Event;
    use yii\helpers\ArrayHelper;
    use yii\helpers\VarDumper;
    use yii\web\Controller;
    use sdelfi\freekassa\actions\ResultAction;
    use sdelfi\freekassa\events\GatewayEvent;
    use sdelfi\freekassa\Merchant;
    
    class PerfectMoneyController extends Controller
    {
        public $enableCsrfValidation = false;
    
        protected $componentName = 'freeKassa';
    
        public function init()
        {
            parent::init();
            /** @var Api $pm */
            $freeKassa = \Yii::$app->get($this->componentName);
            $freeKassa->on(GatewayEvent::EVENT_PAYMENT_REQUEST, [$this, 'handlePaymentRequest']);
            $freeKassa->on(GatewayEvent::EVENT_PAYMENT_SUCCESS, [$this, 'handlePaymentSuccess']);
        }
    
        public function actions()
        {
            return [
                'result' => [
                    'class' => ResultAction::className(),
                    'componentName' => $this->componentName,
                    'redirectUrl' => ['/site/index'],
                    'sendConfirmationResponse' => true
                ],
                'success' => [
                    'class' => ResultAction::className(),
                    'componentName' => $this->componentName,
                    'redirectUrl' => ['/site/index'],
                    'silent' => true,
                    'sendConfirmationResponse' => false
                ],
                'failure' => [
                    'class' => ResultAction::className(),
                    'componentName' => $this->componentName,
                    'redirectUrl' => ['/site/index'],
                    'silent' => true,
                    'sendConfirmationResponse' => false
                ]
           ];
        }
    
        /**
         * @param GatewayEvent $event
         * @return bool
         */
        public function handlePaymentRequest($event)
        {
            $invoice = Invoice::findOne(ArrayHelper::getValue($event->gatewayData, 'MERCHANT_ORDER_ID'));
    
            if (!$invoice instanceof Invoice ||
                $invoice->status != Invoice::STATUS_NEW ||
                ArrayHelper::getValue($event->gatewayData, 'AMOUNT') != $invoice->amount ||
                ArrayHelper::getValue($event->gatewayData, 'MERCHANT_ID') != \Yii::$app->get($this->componentName)->merchantId
            )
                return;
    
            $invoice->debugData = VarDumper::dumpAsString($event->gatewayData);
            $event->invoice = $invoice;
            $event->handled = true;
        }
    
        /**
         * @param GatewayEvent $event
         * @return bool
         */
        public function handlePaymentSuccess($event)
        {
            $invoice = $event->invoice;
            
            // TODO: invoice processing goes here 
        }
    }

## Licence ##

MIT
    
## Links ##

* [Official site](http://yiidreamteam.com/yii2/free-kassa)
* [Source code on GitHub](https://github.com/sdelfi/yii2-free-kassa)
* [Composer package on Packagist](https://packagist.org/packages/sdelfi/yii2-free-kassa)
* [FreeKassa service](http://yiidreamteam.com/link/free-kassa)
