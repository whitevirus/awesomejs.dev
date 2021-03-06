<script>
import gql from 'graphql-tag'
import { pkgFragment } from './fragments'
import { useMutation } from '@vue/apollo-composable'
import { useSelectNextProposal, removeProposalFromCache } from '@/util/proposal'

export default {
  props: {
    projectTypeId: {
      type: String,
      required: true,
    },

    proposal: {
      type: Object,
      required: true,
    },
  },

  setup (props) {
    // Action
    const { mutate } = useMutation(gql`
      mutation ApprovePackageProposal ($input: ApprovePackageProposalInput!) {
        approvePackageProposal (input: $input) {
          ...pkg
        }
      }
      ${pkgFragment}
    `, () => {
      // Save current proposal ID as it will change since we immediately select another proposal
      const currentProposalId = props.proposal.id
      return {
        variables: {
          input: {
            proposalId: currentProposalId,
          },
        },
        update: (cache, { data: { approvePackageProposal } }) => {
          removeProposalFromCache(cache, props.projectTypeId, currentProposalId)
        },
        optimisticResponse: {
          __typename: 'Mutation',
          approvePackageProposal: {
            __typename: 'Package',
            ...props.proposal,
          },
        },
      }
    })

    const { selectNext } = useSelectNextProposal()

    async function approve () {
      await selectNext(props.projectTypeId, props.proposal.id)

      // Approve mutation
      await mutate()
    }

    return {
      approve,
    }
  },
}
</script>

<template>
  <BaseButton
    :disabled="!proposal.repo"
    icon-left="done"
    class="bg-orange-700 hover:bg-orange-800"
    @click="approve()"
  >
    Approve
  </BaseButton>
</template>
