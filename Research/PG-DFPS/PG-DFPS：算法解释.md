$$
\begin{array}{l}
\hline
\textbf{Algorithm 1} \text{ PG-DFPS-Based MA Position Selection} \\
\hline
\textbf{Require:} \text{ Candidate grid } \mathcal A_\Delta\text{; profiles } H(q_k) \text{ and } G(q_k)\text{; channel samples } h(q_k) \text{ and } g(q_k)\text{;} \\
\qquad\quad N, d_{\min}, N_c, \text{ and weights } \omega_H, \omega_G, \omega_\rho. \\
\textbf{Ensure:} \text{ Selected MA position vector } \mathbf p_{\rm PG\text{-}DFPS}. \\
\hline
\phantom{0}1: \text{ Compute } s_{\rm dual}(q_k)=H(q_k)(1-G(q_k)) \text{ for all } q_k\in\mathcal A_\Delta. \\
\phantom{0}2: \text{ Construct candidate pool } \mathcal C \text{ by keeping the } N_c \text{ largest-score grid points.} \\
\phantom{0}3: \text{ Initialize } \mathcal S_0\leftarrow\emptyset. \\
\phantom{0}4: \textbf{for } n=1,\ldots,N \textbf{ do} \\
\phantom{0}5: \qquad \text{Construct } \mathcal C_n(\mathcal S_{n-1}) \text{ according to } \eqref{eq:feasible_candidate_set}. \\
\phantom{0}6: \qquad \text{Evaluate } s_{\rm PG\text{-}DFPS} \bigl(\mathcal S_{n-1}\cup\{q\}\bigr) \text{ for each } q\in\mathcal C_n(\mathcal S_{n-1}) \text{ according to } \eqref{eq:pgdfps_score}. \\
\phantom{0}7: \qquad \text{Select } q_n^\star \text{ according to } \eqref{eq:next_position_selection}. \\
\phantom{0}8: \qquad \text{Update } \mathcal S_n\leftarrow \mathcal S_{n-1}\cup\{q_n^\star\}. \\
\phantom{0}9: \textbf{end for} \\
10: \text{ Sort } \mathcal S_N \text{ in ascending order and form } \mathbf p_{\rm PG\text{-}DFPS}. \\
\hline
\end{array}
$$
