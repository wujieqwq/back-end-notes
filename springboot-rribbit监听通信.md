# RRiBbit：开源事件总线EventBus框架
RRiBbit可以作为事件总线Eventbus, 能够让组件之间进行双向通讯，支持远程功能，实现失败恢复 负载平衡， SSL/TLS等支持，这也称为请求-响应总线(Request-Response-Bus).

## 未使用事件总线
```
public class OrderPage {
 
        private PaymentService paymentService;
        private UserService userService;
         
        public void submitOrder() {
                Integer userId = 1;
                BigDecimal amount = BigDecimal.TEN;
                 
                //依赖支付模块的支付服务
                paymentService.doPayment(userId, amount);
                //依赖用户模块中的用户服务
                userService.registerPayment(userId, amount);
        }
}
 
 
public class PaymentService {
 
        private MailService mailService;
         
        public void doPayment(Integer userId, BigDecimal amount) {
                //Do payment...
                mailService.sendPaymentEmail(userId, amount);
        }
}
 
 
public class UserService {
 
        public String getEmailAddress(Integer userId) {
                return "foo@bar.com";
        }
         
        public void registerPayment(Integer userId, BigDecimal amount) {
                //Register payment in database...
        }
}
 
 
public class MailService {
 
        private UserService userService;
         
        public void sendPaymentEmail(Integer userId, BigDecimal amount) {
                String emailAddress = userService.getEmailAddress(userId);
                //Send email...
        }
}
```
## 使用事件总线
```
public class OrderPage {
 
        private RequestResponseBus rrb;
         
        public void submitOrder() {
                Integer userId = 1;
                BigDecimal amount = BigDecimal.TEN;
                 
                 //发送消息或事件
                rrb.send("doPayment", userId, amount);
                rrb.send("registerPayment", userId, amount);
        }
}
 
 
public class PaymentService {
 
        private RequestResponseBus rrb;
         
        @Listener(hint="doPayment")
        public void doPayment(Integer userId, BigDecimal amount) {
                //Do payment...
                rrb.send("sendPaymentEmail", userId, amount);
        }
}
 
 
public class UserService {
 
        @Listener(hint="getUserEmail")
        public String getEmailAddress(Integer userId) {
                return "foo@bar.com";
        }
         
        @Listener(hint="registerPayment")
        public void registerPayment(Integer userId, BigDecimal amount) {
                //Register payment in database...
        }
}
 
 
public class MailService {
 
        private RequestResponseBus rrb;
         
        @Listener(hint="sendPaymentEmail")
        public void sendPaymentEmail(Integer userId, BigDecimal amount) {
                String emailAddress = rrb.send("getUserEmail", userId);
                //Send email...
        }
}
```
