# sta-tra
Use basic statistical tools to execute buys/sells using Coinbase API. Zero, a brief disclaimer and a brief word of warning all users should be aware of (although it probably won't affect any given user). First install dependencies (section 1), get yourself some API keys from coinbase (section 2), check out the 'To Run' instructions (section 3). If you want to ssh into DigitalOcean to run on the cloud, see section 4, and if you are trying to get stuff running on various operating systems, check out our tips in section 5. If you want to know the mathematical particulars of how decisions are made, check out section 6. Goals for the future are in section 7.

# 0. A word of warning

First, a liability disclaimer. This is the author speaking in first person instead of the royal "we" used throughout the remainder of the document, because this is personal, yo. This software is for novelty use only. I am not a professional programmer, I am a hobbyist, although I have been programming for a long time; so if you lose tens of thousands of dollars, or even just a single month's rent, because you cloned a random amateur author's repo from github, your problems in life include poor decision making skills for which I am not responsible. I make no claims about the robustness of this code nor the profitability of this code. This is a fun pet project for myself and for my friends, all of whom I know would never invest more than they could lose; this is NOT a professional financial institution's trading software. I would be delighted if people started using it, and I would be even more delighted if people started making money from it, but as the primary contributor, I am making NO claims about this code's reliability, robustness, profitability, or security. After all, you never know what mistakes you've made until they've already come crashing down on your head, and I'm a grad student, I'm not a fiduciary.

Second, a **security notice** so please read! If you receive an insecure platform warning while running this software, please take a screen shot and send it to us, and STOP running the code on your machine. These warnings are associated with vulnerabilities to Man-in-the-Middle attacks, and if they pop up it is not a good idea to continue with the code until the issue is resolved. To be clear, the warning does not mean you have been attacked. It means that there is a bug in the code that could allow for an attack. So keep calm, take a screenshot, send it to us, and suspend your trading activity with sta-tra until it's resolved.

# 1. Dependencies

We use the coinbase python library, the json library, the requests library, and scipy.

What's worked for me is to use the following:

        sudo apt-get install git python-scipy python-pip libffi-dev libssl-dev
        sudo pip install pyopenssl ndg-httpsclient pyasn1 coinbase pbkdf2

Sometimes pip gives me trouble and it appears to be resolved by adding `-H` to the pip command above. Also, unless you plan on use `nohup` to run the code continuously, you will also want to include a `sudo apt-get install screen` (although most distributions of ubuntu already have `screen` installed). 

# 2. Coinbase API keys

I don't recommend that you enable any API keys ever, because that's a surefire way to accidentally lose all your money. But if you want to run my code, you gotta. I strongly recommend enabling 2-factor authentication before doing so (not that 2FA will help if someone gets ahold of your keys...) This code will encrypt those keys before storing them locally, but merely having API keys enabled is technically a security risk. Be forewarned.

Go to Coinbase, log in, click on Settings on the sidebar on the left. Go to the API Access tab and then click "+ New API Key." The key you create needs the following stuff enabled:

        wallet:accounts:read 
        wallet:accounts:update 
        wallet:buys:create 
        wallet:buys:read 
        wallet:payment-methods:read 
        wallet:sells:create 
        wallet:sells:read

After they are created, you will run the code below (see the next section) and copy-paste them into the terminal. That is the ONLY time you will have the API keys available for viewing or stealing, and immediately after they are pasted into the terminal, they will be encrypted and stored.

# 3. To run the first time:

Download, run Oracle.py with `python Oracle.py` once to start and then set up `Oracle.py` to run once an hour using `cron` or Windows Task Manager (see below). Start a new screen by typing `screen` (this will allow us to keep `Trader.py` running continuously after we exit the terminal... see the end of this section for information on that). Run Trader.py with `python Trader.py`.

First time users will be prompted for their username, then their API keys, and then prompted for a password. The code will ask you for some user preferences: 

1. What percentage change in bitcoin price should trigger an action? For example, if you want to rebalance if the price moves by more than 3%, then your input should be 0.03. I usually use 0.025; in order to clear fees, you must use values above 0.0201. Note that although this number must be above 0.0201, it can be as large as you want: setting this number to 3.0 will require the price to increase to triple its present value or reduce to 1/3 of its present value before we take any actions, for example. (Suggested value: 0.025 or 0.05)
2. What confidence level do we want to use when drawing our trendlines? For example, if you want 99% confidence intervals, then your input should be 0.99. I usually use 0.99, but this is largely a cosmetic choice from our experiences. Please note this number must be *strictly* between 0.0 and 1.0. Further note that teh closer the number is to 1.0, the wider your confidence interval will be. There is no real harm in choosing a small number like 0.25, but picking a very very large number like 0.999999999 runs the risk of throwing some numerical errors. (Suggested value: 0.99)
3. Of all the bitcoin accounts/wallets that Coinbase has on file for you, which bitcoin account/wallet on Coinbase would you like to trade with? I have a separate bitcoin account/wallet set up on Coinbase specifically for trading.
4. Of all the payment methods that Coinbase has on file for you, which payment method would you like to use? I strongly recommend that you use the USD Wallet in order to ensure "instant" transactions and to prevent overdraft fees from a bank if the code goes wonky. Besides, since the code works continuously (every few seconds, actually), anything less than instant transactions is a recipe for disaster, so ... just use the USD wallet.
5. Of all the bitcoin in your selected account/wallet, how much do you want to be actively trading with? We can verify this account balance with Coinbase, so you can be a little sloppy with this number without getting totally fucked.  (Suggestion: don't trade with all the bitcoin in your trading wallet)
6. Of your USD funds from your payment method, how much are you willing to gamble/flush down the toilet with this code? We cannot verify this number unless you use the USD wallet (and so we don't verify at all in case a user wants to use a bank account or a credit card instead of the USD wallet), so we recommend very carefully answering. If you provide an answer that will cause you to overdraft or get your credit card declined, that's your problem, not ours. This is why we prefer using the Coinbase USD Wallet if possible. (Suggestion: don't trade with all the USD in your USD wallet).


Use `ctrl+a, d` to detach from the screen keeping `Trader.py` running, and then close your terminal with `exit`.  


After that, everything should take off. As a side note, to elaborate on number 6 above, if you do decide to use credit or debit card transactions (which, as of August 1 of 2016, can be instant transactions)...  then we have no way of verifying your account balance through Coinbase. The code could easily make a buy action (or many) that maxes your credit card, or drains your bank account. Please use the USD wallet.


# 4. SSH Keys and DigitalOcean

I recommend running this on some sort of cloud based service (AWS or DigitalOcean or something). I spun up a droplet using instructions [from here](https://www.digitalocean.com/community/tutorials/how-to-create-your-first-digitalocean-droplet-virtual-server), generated my SSH keys using

        ssh-keygen -t rsa

I usually use the default folder, and I generate passphrases using LastPass. To get my SSH keys I use

        cat ~/.ssh/id_rsa.pub

which I then copy-paste into the DigitalOcean SSH key prompt. We use

        ssh root@ip.address.here
        
to ssh into the droplet as root, and then we create a new user, add it to the sudo group, and generate ssh keys for that new user. Then rather than loggin in as root, log in as the local user.


# 5. Different Operating Systems

I've had a surprising amount of difficulty getting this going in different settings. I've tried to summarize the methods that have worked well below:

## 5a. Running with Windows 10

This has entirely been developed in Ubuntu (precise and trusty usually) so I have no idea how well it will port to windows. I also am under the impression that numpy and scipy are both a pain in the ass with Windows. So, I dunno, dual boot yourself some Ubuntu or use some Crouton for this, or use Virtualbox in windows to run Ubuntu (see below).

## 5b. With Virtualbox

Head over to Ubuntu.com and grab yourself an Ubuntu 14.04 iso file and use Virtualbox to set up a virtual ubuntu box. Install the dependencies below, and then install guest additions. You can use the command line to do so with `sudo apt-get install virtualbox-guest-dkms` but this didn't work very well for me to get bidirectional clipboard and resolution resizing working. I had to (in the virtualbox window) click on Devices, then Insert Guest Additions CD Image. After installing, rebooting virtualbox got the clipboard and resolution working.  I then installed all the dependencies below, cloned this git repository, added Oracle to crontab, ran it once, and then started running Trader.py

## 5c. Running with Crouton

### Installing crouton:

I have had some success with unity, but I primarily use xfce. To start using crouton on your Chromebook: after entering developer mode and downloading crouton, I use:

        sudo sh ~/Downloads/crouton -r trusty -t xfce,xiwi,extension,chrome
        sudo start xfce4
        
Make sure you are up to date with this:

        sudo sh ~/Downloads/crouton -n trusty -u
        
Despite that it seems like trusty tahr works better on my chromebook than precise pangolin, and xfce is the only version I've gotten to work so far, it's still quite janky/iffy using ctrl-alt-shift-forward/back to switch operating systems. 

### Running Oracle.py with cron in crouton

Now to get cron working.... According to [dnschneid](https://github.com/dnschneid/crouton/wiki/Setting-Up-Cron-Job) crouton needs some massaging to run cron. To get cron working with crouton, I use 

        sudo gedit /etc/rc.local

and add the line `exec cron` before the line `exit 0:` which will get cron running. Dnschneid says user level process don't work, so to add jobs we need to add them to cron at the root level. I use the command

        sudo gedit /etc/crontab

and add the following line to the bottom of my crontab file:

        1 * * * * root /usr/bin/python /path/to/scripts/Oracle.py
        

# 6. Mechanics

This section describes how the code actually works. It's broken into a few subsections: password management (because there's no good reason to save your API keys as a plaintext file on a computer or, worse, on a server), fundamental dynamics (to describe exactly what's going on with the price), trend-based triggers (to describe how we compute all this), and bookkeeping.

## Password management, logging in, encryption:

All this is handled by AESCrypt.py and API_Key_Manager.py.

A file, `key_manager.txt`, with user login info will be kept for encryption purposes. Each user's username appears, as well as the salt for hashing their password, and their hashed password (raw passwords are not stored). Additionally, two more salts are stored in this file for hashing the same password to get AES encryption/decryption keys. Why two? Coinbase API keys come in pairs (the key and the secret).

If the username does not exist in the directory, a password is registered to them, the program prompts the user for API keys, generates salt for encrypting each key, and uses the password to encrypt those keys with the appropriate salt. The password is also hashed with some known salt, and the username, password salt, hashed password, and encryption key salts are all written to `key_manager.txt`. Thus, if a username exists in the directory, salts and hashed passwords should also be in the directory. After all that, the API keys are returned.

If the username exists and the provided password concatenated with the salt in `key_manager.txt` hashes to the hashed password in `key_manager.txt`, the user is validated and the keys for decrypting the API info are generated, and the API keys are returned to the user.


## Fundamental Dynamics

First problem: "How and when do we decide to issue `buy` or `sell` actions?" First, when the user logs in for the first time, they set the percentage change in price that triggers an action (question 1 in Section "To Run"), maybe call this `p`.  Second, We compute the minimum of all the unmatched `buy` actions on record, say `min_old_buy_prices`, and the maximum of all unmatched `sell` actions on record, say `max_old_sell_prices`. Third, we compute the trend-based triggers, say `trend_buy_trig` and `trend_sell_trig` as described below. Then we do something akin to this:
        
        if sell_price >= min(min_old_buy_prices*(1.0 + p), max(trend_sell_trig, max_old_sell_prices*(1.0 + p)))
                # Then we sell
        if buy_price <= max(max_old_sell_prices/(1.0 + p), min(trend_buy_trig, min_old_buy_prices/(1.0 + p)))
                # Then we buy

Note: `trend_buy_trig` and `trend_sell_trig` are determined with the market information using `Oracle.py` completely independent of past trades. The idea of the above is: regardless of the market trend and history, if a chance is available to take some profit by matching an old `buy` or `sell` action with a new, dual action, we should take it. On the other hand, if we have no historical information, we should judge the recent trend of prices before making a move.

## Trend-based triggers

The file `Oracle.py` should be run once an hour using `cron` or whatever. It pulls hourly historical pricing information from Coinbase. We determine a preferred timescale that maximizes a normalized signal-to-noise ratio (SNR) for the log of the price in `_find_good_sample_size`. That sounds technical but we do the following: for each possible timescale, `i` hours (integer), compute the average of the past `i` hours of the natural log of the price and call this `y_mean`. Also compute and the (unbiased) standard deviation of the past `i` hours, `y_stdev`, and define the SNR `snr = y_mean*sqrt(i)/y_stdev` and we choose the `i` that maximizes this ratio. Then, in `_get_linear_trend` with our preferred timescale in hand, we sample the last `i` hours of pricing information and we find the OLS best-fit line, say `log(trend_price) - y_mean = best_fit_slope*(time - t_mean)` . Then we hypothesize that deviations from this line, (residual, say `z_i = abs(log(price(i)) - log(trend_price))`) are i.i.d. zero-mean normal random variables (this assumption is false in general, and will be improved eventually).  From this and using `y_stdev`, we can generate a `100(1-alpha/2)` percent confidence interval, which we can apply to the trend to get an upper and lower bound on price. These parameters (`y_mean`, `t_mean`, `best_fit_slope`, `y_stdev`, and the residuals) are used in `Trader.py`; we call the upper and lower bounds of this window `trend_buy_trig` and `trend_sell_trig`.

## Bookkeeping

We have `buy` actions and `sell` actions that take place on a timeline. We want to link actions into bets with a low buy and high sell. These bets will  consisting of an ordered pair `(action1, action2)`. The timestamp of `action1` always occurs before the timestamp of `action2`. If `action1` is a `buy` action then `action2` must be a `sell` action such that the net profit in USD is positive. If `action2` is a `sell` action then `action1` must be a `buy` action such that the net profit in BTC is positive. If a pair of actions are linked in such an ordered pair, so we call these actions "paired."

As we issue actions to Coinbase and then receive confirmation of those actions, the results are added to a `Buy_Q` and a `Sell_Q` in chronological order. After an action is added to these queues, we look for possible pairs/bets following first-in-first-out rules: we will pair the earliest chronologically occurring action in either `Buy_Q` or `Sell_Q` that has a corresponding action in the opposite queue satisfying the profit condition. We de-queue the to-be-paired actions, store them into an ordered pair of the form `(action1, action2)`, and append the resulting ordered pair to a file, say `bet_history.csv` (these are resolved bets, no need to keep their information liquid).

If an action remains in the `Buy_Q` or the `Sell_Q` for a long time, this means that the price hasn't allowed this action to be paired. This corresponds to buying high before a price drop, or selling low before a price rise. These are bad moves that we need to remember, historically, so that we can try to recover from epic bad decisions from the past. Hence, we need our current `Buy_Q` and `Sell_Q` to also be written to file after each time they are updated, say `Buy_Q.csv` and `Sell_Q.csv`. This way, each time we load the program, we pick up where we left off.

# 7. Future implementations 

Super long-term, we wwould like to develop some neural networks that are trained to respond to the time series. More short-term and attainable goals include (in no particular order) (i) more complicated buy/sell strategies (maybe), (ii) more statistically sophisticated time series methods of analysis (we're thinking to BLUE this), (iii) we want the code to send text messages or e-mails when actions are completed, (iv) amounts to buy/sell will be modified once we have some probabilities estimated so we can start using Kelly betting, and (v) better-than-hourly historical pricing scheme, possibly pulling and logging price every minute or every 15 minutes or whatever and storing it into a database.
