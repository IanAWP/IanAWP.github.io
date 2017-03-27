# Parallel Processing

The final optimization in this series exists in somewhat of a different dimension to the other things we’ve tried.  One of the stated goals for this work was to be able to produce high quality tiles on demand – meaning that we’ve focused on improving the performance of generating a single tile.

Given that I have been using the *world_resample* dataset as my benchmark, and given that I’ve been tiling the whole dataset, I thought I’d parallelize the process to see what sort of improvements I can get.

Once again not much has to change.  Our tile byte array can’t be shared by multiple threads, so it either needs to be declared inside the loop, or needs to be made thread local.

ThreadLocal<byte[]> terrainTileLocal = new ThreadLocal<byte[]>(() => new byte[(bytes + 2)]);

We need to restrict the parallel processing to within the same zoom level. Lower zooms depend on higher zooms both because of the tile saving optimisation, and because of the child tile byte.  I’ve chosen simply to use:

Parallel.For(0, dx + 1, (easterlyOffset) => {

for each dimension. Finally, the progress reporting needs to change

…And bob’s your uncle!

<table>
  <tr>
    <td>Method</td>
    <td>Average Time</td>
  </tr>
  <tr>
    <td>naive</td>
    <td>00:31.9</td>
  </tr>
  <tr>
    <td>write saving</td>
    <td>00:23.9</td>
  </tr>
  <tr>
    <td>tile saving</td>
    <td>00:20.9</td>
  </tr>
  <tr>
    <td>Block @ Max Zoom</td>
    <td>00:12.1</td>
  </tr>
  <tr>
    <td>Parallel</td>
    <td>00:06.1</td>
  </tr>
</table>

So that's that for now.  In the next post I'll give a quick summary of what's left undone in this implementation, and introduce the quantized mesh terrain format, which I'm interested in supporting with this project in future.