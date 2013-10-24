# Non-Relational Models > Model Hooks

Agile Toolkit implements `hook()` and `addHook()` on a very low level. Any object may define `hook()` and other objects can place call-backs by previously calling `addHook()`.

Model defines 8 hooks. Four of them are executed before "load", "unload", "delete" and "save" and other four are executed after. beforeLoad, beforeSave and beforeDelete are being passed 2nd argument which is optional "id". Like any other hook, first argument is going to be object hook was placed on. In our case - The Model.

By using those callbacks it's possible to implement additional controllers such as timestamps, owner-stamps, audit logs, field encryption. It's also great that any object can use those callbacks, even model itself.

    class Model_User extends Model {
      function init(){
        parent::init();

        $this->add('Controller_Audit');

        $this->addField('name');
        $this->addField('email');
        $this->addField('password')->display('form'=>'password');

        $this->addHook('beforeSave',function($m){

          // Validate email before saving
          if(!filter_var($m['email'], FILTER_VALIDATE_EMAIL))
            throw $m->exception('Invalid email')->setField('email');

          // Encrypt password before save
          $m['password']=$m->api->auth->encryptPassword($m['password'],$m['email']);
        });
      }
    }

    class Controller_Audit extends AbstractController {
      function init(){
        parent::init();
        $this->owner->addField('created_dts')->type('datetime')
          ->defaultValue(date('Y-m-d H:i:s'))->system(true);
        $this->owner->addField('updated_dts')->type('datetime');
        $this->owner->addHook('beforeSave',$this);
      } 
      function beforeSave($m){
        if($m->loaded())$m['updated_dts']=date('Y-m-d H:i:s');
      }
    }
As you see, hooks can be used for validation, data conversion, timestamps and hooks can be added in many different ways.