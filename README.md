# From TrackMate to diffusion coefficients

The following code allows the user to use a set of TrackMate generated *csv* tracks to compute the MSD curve and diffusion coefficient associated to each track. The experimental time step `dt` between each frame must be set by the user. I use `Pandas` to turn the csv files into a `DataFrame`. I then perform a loop over the track IDs to recover the positions of the constitutive spots and compute a mean square displacement for each track.

### Filter on the number of frames

The user may want to filter out tracks that are very short, i.e. these tracks may result from the tracking of background noise. The minimum number of frames is generally set to 5 or more. One may also want to get rid of very long tracks, which may represent the tracking of dust in the original movie. 

### Computation of the MSD curves

Once the coordinates of each spot that constitutes a trajectory are known, we can compute the MSD for each chosen timelag $n \Delta \tau$ : 

$$ \rho_n = \frac{1}{N - n} \sum_{i=0}^{N-n-1}\left[(x_{n+i} - x_i)^2 + (y_{n+i} - y_i)^2\right] \qquad n=1,...,N-1 $$

For a given trajectory, we can compute $N-1$ such $\rho_n$, which we will store in an array called `rhon`. This is the MSD curve associated to a given track. For free diffusion, $\rho$ is expected to be linear in $n \Delta \tau$. In practise, $\rho$ will be modelled with a function in power $\alpha$: $\rho(n\Delta \tau) = 4 D (n\Delta \tau)^\alpha$. I use the package `lmfit` to perform the fit. Each fit parameter $D$ and $\alpha$ can be bounded. 

The fit of a MSD curve should not be performed over all tracks, as the accuracy of the MSD decreases as the timelag increases. In the same way, localization uncertainty is large for small timelags. I will perform the fits over 30 % of the data points. Later on, once the free diffusion tracks are identified, I will perform the fit using a more optimal approach described by Michalet (2010). We can compute a $R^2$ coefficient of determination and get rid of the tracks for which the fit of the MSD curve has $R^2$ below a user-defined threshold. 

For each track, the value of $\alpha$, $D$, the total number of frames $N$ and the confinement ratio (defined as the end-to-end distance divided by the total displacement) are stored in the `alphaD` matrix. These quantities will be used to perform an unsupervised cluster classification. 
