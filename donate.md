---
title: Donate
date: 2020-04-05
layout: default
comments: true
published: true
description: Donate
keywords: donate paypal stripe 
---

# Donate

If you think my blog and support is useful for you, 
I would appreciate a small donation:

* **[PayPal](https://www.paypal.me/dopitz/5)**

You can **buy** all Slim articles (> 200 pages) bundled into a eBook. 

<div style="text-align: center;">
<img src="https://user-images.githubusercontent.com/781074/92960025-27871c00-f46d-11ea-87be-3b460f4edeec.png">
</div>

<script src="https://cdn.jsdelivr.net/npm/sweetalert2@10"></script>

<div id="paypal-button-container"></div>
<script src="https://www.paypal.com/sdk/js?client-id=AdgjzHMB-hvxJGnVoxlkeGzRwFlItWRfDCLLIeHMwqmi6TpUU24NlOyWjbCmPiHTzpekYxqgXA1IyJYz&currency=EUR" data-sdk-integration-source="button-factory"></script>
<script>
    paypal.Buttons({
        style: {
            shape: 'rect',
            color: 'gold',
            layout: 'vertical',
            label: 'paypal',

        },
        createOrder: function(data, actions) {
            return actions.order.create({
                purchase_units: [{
                    amount: {
                        value: '9'
                    }
                }]
            });
        },
        onApprove: function(data, actions) {
            return actions.order.capture().then(function(details) {
                //alert('Transaction completed by ' + details.payer.name.given_name + '!');
                Swal.fire({
                    icon: 'success',
                    text: 'Transaction completed. Thank you! Please check your inbox. If you did not receive your activation email, please check your spam filter settings and your spam folders.',
                });
            });
        },
        onCancel: function (data, actions) {
            Swal.fire({
                icon: 'error',
                text: 'Transaction canceled. Please try it again!',
            });
        }
    }).render('#paypal-button-container');
</script>
<br>

Please note: The e-mail delivery can take up to 12 hours.
If you have not received your ebook, please check your junk mail folder.
