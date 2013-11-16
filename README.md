A fork of Xelerance's fork of xl2tpd.

I setup a xl2tpd server on Amazon EC2, and it worked beautifully with iOS/MacOS/Windows 7, but always disconnects Android L2TP client after several minutes.

After nearly a month of debugging effort, I still cannot give a definitive answer to the problem, but there is a workaround.

In the original issue, the Android client appears to always request tunnel twice - "peer requested tunnel xxx twice", but it does not happen on other clients (MacOS, Macbook, Windows 7, iOS).

In the source code, xl2tpd kills a tunnel if retransmission counter reaches certain threshold, it logs a message saying "Maximum retries exceeded for tunnel xxx" then hangs up on PPP connection.

But the problem is: for whatever reason, the tunnel is the __actively used__ tunnel, so hanging it up means terminating Android's L2TP connection.

So I ended up forking xl2tpd version 1.3.1 to https://github.com/HouzuoGuo/xl2tpd in branch `1.3.1`. With my fixes, xl2tpd no longer kills tunnel on "maximum retries exceeded", it simply logs a message and moves on.

All clients are now happy, Android no longer disconnects and the same configuration still works beautifully on MacOS/iOS/Windows 7.

By the way, xl2tpd 1.3.2 has been released, but according to my tests, it does not work with Android at all:

- Scheduler responsible for calculating `select()` timeout yields a timeout too short (sub second), resulting in lots of network timeouts and Android L2TP connection cannot be established in-time.
- Even if the `select()` timeout is manually changed (to 5 or 10 seconds), the "peer requested tunnel xxx twice" problem not only exists, but it gets worse - Android cannot establish a connection at all.
