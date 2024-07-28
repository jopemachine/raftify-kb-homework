# FLP Theorem
FLP Theorem(Impossibility) states that all of `Liveness`, `Safety`, `Fault Tolerance` can't be achienved. In short, every consensus algorithm have to select which attribute to achieve.

## Safety
All non-faulty participants must agree on the same state, and that state must be valid. Simply put, correct nodes do not reach an incorrect consensus.

## Liveness
The system must eventually reach an agreement on some state, and all non-faulty participants must achieve consensus. In essence, correct nodes must eventually agree.

## Fault Tolerance
The system must handle a certain number of node failures without compromising `Safety` and `Liveness`. However, due to the FLP theorem, achieving `Fault Tolerance` while maintaining both `Safety` and `Liveness` in an asynchronous system is impossible.
